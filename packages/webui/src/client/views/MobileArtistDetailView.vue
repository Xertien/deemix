<script setup lang="ts">
import { formatArtistData, getArtistData } from "@/data/artist";
import { sendAddToQueue } from "@/utils/downloads";
import { computed, onMounted, ref, watch } from "vue";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const router = useRouter();

const artistId = computed(() => route.params.id?.toString() ?? "");
const isLoading = ref(true);
const errorMessage = ref("");
const artistName = ref("");
const artistPicture = ref("");
const artistReleases = ref<Record<string, any[]>>({});

async function loadArtist() {
	const id = artistId.value;
	if (!id) {
		errorMessage.value = "Artiste introuvable.";
		return;
	}

	isLoading.value = true;
	errorMessage.value = "";

	try {
		const data = await getArtistData(id);
		const { artistName: name, artistPictureXL, artistReleases: releases } =
			formatArtistData(data);

		artistName.value = name || "Artiste";
		artistPicture.value =
			artistPictureXL ||
			"https://e-cdns-images.dzcdn.net/images/artist//1000x1000-000000-80-0-0.jpg";
		artistReleases.value = releases || {};
	} catch (error) {
		console.error(error);
		errorMessage.value =
			"Impossible de charger les informations de l'artiste pour le moment.";
	} finally {
		isLoading.value = false;
	}
}

function addArtistToQueue() {
	sendAddToQueue(`https://www.deezer.com/artist/${artistId.value}`);
}

function openAlbum(releaseId: string) {
	router.push({ name: "MobileAlbumDetail", params: { id: releaseId } });
}

watch(
	() => route.params.id,
	() => {
		loadArtist();
	}
);

onMounted(() => {
	loadArtist();
});
</script>

<template>
	<section class="mobile-artist">
		<div class="mobile-card mobile-card--hero">
			<p class="mobile-eyebrow">Artiste</p>
			<h1>{{ artistName }}</h1>

			<div v-if="artistPicture" class="mobile-artist__picture">
				<img :src="artistPicture" :alt="artistName" />
			</div>

			<button class="mobile-link" type="button" @click="addArtistToQueue">
				Ajouter tous les titres de l'artiste
			</button>
		</div>

		<section v-if="isLoading" class="mobile-card mobile-card--notice">
			Chargement des sorties…
		</section>

		<p v-if="errorMessage" class="mobile-error">{{ errorMessage }}</p>

		<section
			v-for="(releases, type) in artistReleases"
			:key="type"
			class="mobile-card mobile-card--list"
		>
			<div class="mobile-card__header">
				<h2>{{ type }}</h2>
				<span>{{ releases.length }}</span>
			</div>

			<ul class="mobile-artist__releases">
				<li
					v-for="release in releases"
					:key="release.releaseID"
					class="mobile-artist__release"
				>
					<button
						type="button"
						class="mobile-artist__release-button"
						@click="openAlbum(release.releaseID)"
					>
						<img
							:src="release.releaseCover"
							:alt="release.releaseTitle"
							width="64"
							height="64"
						/>
						<div>
							<p class="mobile-search__title">{{ release.releaseTitle }}</p>
							<p class="mobile-search__subtitle">
								{{ release.releaseDate }} •
								{{ release.releaseTracksNumber }} titres
							</p>
						</div>
					</button>

					<button
						type="button"
						class="mobile-chip"
						@click.stop="sendAddToQueue(release.releaseLink)"
					>
						Ajouter
					</button>
				</li>
			</ul>
		</section>
	</section>
</template>

<style scoped>
.mobile-artist {
	display: flex;
	flex-direction: column;
	gap: 1rem;
	padding-bottom: 4rem;
}

.mobile-artist__picture {
	width: 160px;
	height: 160px;
	border-radius: 50%;
	overflow: hidden;
	margin: 0.75rem 0;
}

.mobile-artist__picture img {
	width: 100%;
	height: 100%;
	object-fit: cover;
}

.mobile-artist__releases {
	display: flex;
	flex-direction: column;
	gap: 0.75rem;
}

.mobile-artist__release {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 0.75rem;
}

.mobile-artist__release-button {
	display: flex;
	align-items: center;
	gap: 0.75rem;
	background: none;
	border: none;
	text-align: left;
	color: inherit;
	width: 100%;
}

.mobile-artist__release-button img {
	border-radius: 12px;
}

.mobile-error {
	color: #ff9b9b;
}
</style>

