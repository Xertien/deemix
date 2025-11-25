<script setup lang="ts">
import { fetchData } from "@/utils/api-utils";
import { computed, onMounted, onUnmounted, ref } from "vue";

interface QueueEntry {
	uuid: string;
	title: string;
	artist?: string;
	progress?: number;
	size?: number;
	downloaded?: number;
	status?: string;
	cover?: string;
}

interface QueueSnapshot {
	current?: QueueEntry | null;
	queue?: Record<string, QueueEntry>;
	queueOrder?: string[];
}

const queueSnapshot = ref<QueueSnapshot | null>(null);
const isLoading = ref(true);
const errorMessage = ref("");
const lastUpdatedLabel = ref<string>("");
let pollInterval: number | null = null;

async function loadQueue() {
	try {
		const data = await fetchData("getQueue");
		queueSnapshot.value = data;
		errorMessage.value = "";
		lastUpdatedLabel.value = new Date().toLocaleTimeString();
	} catch (error) {
		console.error(error);
		errorMessage.value =
			"Impossible de récupérer la file de téléchargement pour le moment.";
	} finally {
		isLoading.value = false;
	}
}

const currentJob = computed(() => queueSnapshot.value?.current ?? null);
const orderedQueue = computed(() => {
	if (!queueSnapshot.value?.queueOrder || !queueSnapshot.value?.queue) return [];

	return queueSnapshot.value.queueOrder
		.map((uuid) => queueSnapshot.value?.queue?.[uuid])
		.filter((job): job is QueueEntry => Boolean(job));
});

function isFinished(job?: QueueEntry | null) {
	if (!job) return false;

	if (job.status && job.status.toLowerCase().includes("finished")) {
		return true;
	}

	if (job.downloaded && job.size) {
		return job.downloaded === job.size;
	}

	return getProgress(job) >= 100;
}

const upcomingJobs = computed(() => {
	const currentUuid = currentJob.value?.uuid;
	return orderedQueue.value.filter(
		(job) => job.uuid !== currentUuid && !isFinished(job)
	);
});

const finishedJobs = computed(() => {
	if (!queueSnapshot.value?.queue) return [];
	return Object.values(queueSnapshot.value.queue).filter((job) =>
		isFinished(job)
	);
});

function getProgress(job?: QueueEntry | null) {
	if (!job) return 0;
	if (typeof job.progress === "number") {
		return Math.max(0, Math.min(100, Math.round(job.progress)));
	}

	const downloaded = job.downloaded ?? 0;
	const size = job.size ?? 0;
	if (!size) return 0;

	return Math.max(0, Math.min(100, Math.round((downloaded / size) * 100)));
}

function getStatus(job?: QueueEntry | null) {
	if (!job) return "En attente";
	if (job.status) return job.status;

	const progress = getProgress(job);
	if (progress >= 100) return "Téléchargement terminé";
	if (progress > 0) return "Téléchargement en cours";
	return "En file d'attente";
}

function getCounter(job?: QueueEntry | null) {
	if (!job) return "";

	const downloaded = job.downloaded ?? 0;
	const size = job.size ?? 0;
	if (!size) return "";

	return `${downloaded}/${size}`;
}

function refreshQueue() {
	isLoading.value = true;
	return loadQueue();
}

onMounted(() => {
	loadQueue();
	pollInterval = window.setInterval(loadQueue, 5000);
});

onUnmounted(() => {
	if (pollInterval) {
		window.clearInterval(pollInterval);
		pollInterval = null;
	}
});
</script>

<template>
	<section class="mobile-downloads">
		<header class="mobile-downloads__header">
			<div>
				<h1>Téléchargements</h1>
				<p v-if="lastUpdatedLabel" class="mobile-downloads__timestamp">
					Mise à jour {{ lastUpdatedLabel }}
				</p>
			</div>
			<button type="button" class="mobile-chip" @click="refreshQueue">
				Actualiser
			</button>
		</header>

		<p v-if="errorMessage" class="mobile-downloads__error">{{ errorMessage }}</p>

		<section v-if="isLoading" class="mobile-card mobile-card--notice">
			Chargement de la file en cours…
		</section>

		<template v-else>
			<article class="mobile-card mobile-card--hero">
				<h2 v-if="currentJob">Téléchargement actif</h2>
				<h2 v-else>Aucun téléchargement en cours</h2>

				<div v-if="currentJob" class="mobile-downloads__current">
					<div>
						<p class="mobile-downloads__title">{{ currentJob.title }}</p>
						<p class="mobile-downloads__artist">{{ currentJob.artist }}</p>
					</div>

					<p class="mobile-downloads__status">
						{{ getStatus(currentJob) }} {{ getCounter(currentJob) }}
					</p>

					<div class="mobile-progress">
						<div
							class="mobile-progress__bar"
							:style="{ width: `${getProgress(currentJob)}%` }"
						></div>
					</div>
				</div>

				<p v-else class="mobile-downloads__info">
					Ajoutez un lien pour lancer un téléchargement. Il apparaîtra ici
					lorsqu'il sera en cours.
				</p>
			</article>

			<section class="mobile-card mobile-card--list">
				<div class="mobile-card__header">
					<h2>File d'attente</h2>
					<span>{{ upcomingJobs.length }} éléments</span>
				</div>

				<div v-if="!upcomingJobs.length" class="mobile-downloads__info">
					Aucun téléchargement en attente.
				</div>

				<ul v-else class="mobile-downloads__list">
					<li
						v-for="job in upcomingJobs"
						:key="job.uuid"
						class="mobile-downloads__item"
					>
						<div>
							<p class="mobile-downloads__title">{{ job.title }}</p>
							<p class="mobile-downloads__artist">{{ job.artist }}</p>
						</div>
						<div class="mobile-downloads__meta">
							<span>{{ getStatus(job) }}</span>
							<span v-if="getCounter(job)">{{ getCounter(job) }}</span>
						</div>
					</li>
				</ul>
			</section>

			<section class="mobile-card mobile-card--list">
				<div class="mobile-card__header">
					<h2>Terminés récemment</h2>
					<span>{{ finishedJobs.length }}</span>
				</div>

				<div v-if="!finishedJobs.length" class="mobile-downloads__info">
					Aucun historique disponible.
				</div>

				<ul v-else class="mobile-downloads__list mobile-downloads__list--compact">
					<li
						v-for="job in finishedJobs.slice(-5).reverse()"
						:key="job.uuid"
						class="mobile-downloads__item"
					>
						<div>
							<p class="mobile-downloads__title">{{ job.title }}</p>
							<p class="mobile-downloads__artist">{{ job.artist }}</p>
						</div>
						<span class="mobile-downloads__badge">Terminé</span>
					</li>
				</ul>
			</section>
		</template>
	</section>
</template>

<style scoped>
.mobile-downloads {
	display: flex;
	flex-direction: column;
	gap: 1rem;
	padding-bottom: 4rem;
}

.mobile-downloads__header {
	display: flex;
	align-items: center;
	justify-content: space-between;
}

.mobile-downloads__header h1 {
	font-size: 1.5rem;
	margin: 0;
}

.mobile-downloads__timestamp {
	font-size: 0.85rem;
	color: rgba(255, 255, 255, 0.6);
}

.mobile-downloads__error {
	color: #ff9b9b;
	font-size: 0.9rem;
}

.mobile-downloads__current {
	display: flex;
	flex-direction: column;
	gap: 0.4rem;
}

.mobile-downloads__title {
	font-weight: 600;
}

.mobile-downloads__artist {
	font-size: 0.9rem;
	opacity: 0.75;
}

.mobile-downloads__status {
	font-size: 0.85rem;
	text-transform: capitalize;
	opacity: 0.8;
}

.mobile-downloads__info {
	font-size: 0.95rem;
	opacity: 0.8;
}

.mobile-progress {
	height: 6px;
	width: 100%;
	border-radius: 999px;
	background: rgba(255, 255, 255, 0.15);
	overflow: hidden;
}

.mobile-progress__bar {
	height: 100%;
	background: var(--accent-primary, #a46cf5);
	transition: width 0.25s ease;
}

.mobile-card__header span {
	font-size: 0.85rem;
	opacity: 0.7;
}

.mobile-downloads__list {
	display: flex;
	flex-direction: column;
	gap: 0.75rem;
}

.mobile-downloads__item {
	display: flex;
	justify-content: space-between;
	gap: 0.75rem;
}

.mobile-downloads__meta {
	display: flex;
	flex-direction: column;
	gap: 0.2rem;
	font-size: 0.85rem;
	align-items: flex-end;
}

.mobile-downloads__badge {
	border: 1px solid rgba(255, 255, 255, 0.2);
	padding: 0.15rem 0.65rem;
	border-radius: 999px;
	font-size: 0.75rem;
}

.mobile-downloads__list--compact .mobile-downloads__item {
	align-items: center;
}

.mobile-chip {
	border: 1px solid rgba(255, 255, 255, 0.25);
	background: none;
	color: inherit;
	border-radius: 999px;
	padding: 0.35rem 0.9rem;
	font-size: 0.85rem;
}
</style>

