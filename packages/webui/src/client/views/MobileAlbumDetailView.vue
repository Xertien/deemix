<script setup lang="ts">
import { sendAddToQueue } from "@/utils/downloads";
import { convertDuration } from "@/utils/utils";
import { fetchData } from "@/utils/api-utils";
import { computed, onMounted, ref, watch } from "vue";
import { useRoute } from "vue-router";

interface AlbumTrack {
	id: number | string;
	title: string;
	title_version?: string;
	duration: number;
	artist?: { id: string; name: string };
	track_position?: number;
	explicit_lyrics?: boolean;
	link?: string;
	type?: string;
	number?: number;
}

interface AlbumPayload {
	id: string;
	title: string;
	cover_xl: string;
	explicit_lyrics: boolean;
	label: string;
	release_date: string;
	artist: { name: string; id: string };
	tracks: AlbumTrack[];
	nb_tracks: number;
}

const route = useRoute();
const albumId = computed(() => route.params.id?.toString() ?? "");

const isLoading = ref(true);
const errorMessage = ref("");
const albumInfo = ref<AlbumPayload | null>(null);

async function loadAlbum() {
	const id = albumId.value;
	if (!id) {
		errorMessage.value = "Album introuvable.";
		return;
	}

	isLoading.value = true;
	errorMessage.value = "";

	try {
		const data = (await fetchData("getTracklist", {
			type: "album",
			id,
		})) as AlbumPayload;
		albumInfo.value = data;
	} catch (error) {
		console.error(error);
		errorMessage.value =
			"Impossible de charger l'album pour le moment. Réessayez plus tard.";
	} finally {
		isLoading.value = false;
	}
}

function addAlbumToQueue() {
	sendAddToQueue(`https://www.deezer.com/album/${albumId.value}`);
}

function addTrackToQueue(link?: string) {
	if (!link) return;
	sendAddToQueue(link);
}

watch(
	() => route.params.id,
	() => {
		loadAlbum();
	}
);

onMounted(() => {
	loadAlbum();
});
</script>

<template>
	<section class="mobile-album">
		<section class="mobile-card mobile-card--hero" v-if="albumInfo">
			<p class="mobile-eyebrow">Album</p>
			<h1>{{ albumInfo.title }}</h1>
			<p class="mobile-hero__subtitle">
				{{ albumInfo.artist?.name }} •
				{{ albumInfo.nb_tracks }} titres •
				{{ albumInfo.release_date?.substring(0, 10) }}
			</p>

			<img
				v-if="albumInfo.cover_xl"
				:src="albumInfo.cover_xl"
				:alt="albumInfo.title"
				class="mobile-album__cover"
			/>

			<button class="mobile-link" type="button" @click="addAlbumToQueue">
				Télécharger l'album
			</button>
		</section>

		<section v-if="isLoading" class="mobile-card mobile-card--notice">
			Chargement des morceaux…
		</section>

		<p v-if="errorMessage" class="mobile-error">{{ errorMessage }}</p>

		<section
			v-if="albumInfo && albumInfo.tracks?.length"
			class="mobile-card mobile-card--list"
		>
			<h2>Pistes</h2>
			<ul class="mobile-album__tracks">
				<li
					v-for="(track, index) in albumInfo.tracks"
					:key="`${track.id}-${index}`"
					v-if="track.type !== 'disc_separator'"
					class="mobile-album__track"
				>
					<div>
						<p class="mobile-search__title">
							{{ track.track_position || index + 1 }}.
							{{ track.title }}
							<span v-if="track.title_version && track.title_version.length">
								{{ track.title_version }}
							</span>
						</p>
						<p class="mobile-search__subtitle">
							{{ track.artist?.name }} •
							{{ convertDuration(track.duration) }}
						</p>
					</div>
					<button
						type="button"
						class="mobile-chip"
						@click="addTrackToQueue(track.link)"
					>
						Ajouter
					</button>
				</li>

				<li
					v-else
					class="mobile-album__disc"
				>
					Disc {{ track.number }}
				</li>
			</ul>
		</section>
	</section>
</template>

<style scoped>
.mobile-album {
	display: flex;
	flex-direction: column;
	gap: 1rem;
	padding-bottom: 4rem;
}

.mobile-album__cover {
	width: 100%;
	max-width: 320px;
	border-radius: 24px;
	margin: 1rem 0;
	display: block;
}

.mobile-album__tracks {
	display: flex;
	flex-direction: column;
	gap: 0.75rem;
}

.mobile-album__track {
	display: flex;
	justify-content: space-between;
	align-items: center;
	gap: 0.75rem;
}

.mobile-album__disc {
	text-transform: uppercase;
	font-size: 0.85rem;
	opacity: 0.6;
	padding: 0.35rem 0;
}

.mobile-error {
	color: #ff9b9b;
}
</style>

