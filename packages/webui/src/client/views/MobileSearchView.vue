<script setup lang="ts">
import {
	formatAlbums,
	formatArtist,
	formatPlaylist,
	formatSingleTrack,
} from "@/data/search";
import { fetchData } from "@/utils/api-utils";
import { sendAddToQueue } from "@/utils/downloads";
import { computed, reactive, ref } from "vue";

interface MobileSearchResults {
	query: string;
	tracks: ReturnType<typeof formatSingleTrack>[];
	albums: ReturnType<typeof formatAlbums>[];
	artists: ReturnType<typeof formatArtist>[];
	playlists: ReturnType<typeof formatPlaylist>[];
}

const searchTerm = ref("");
const isSearching = ref(false);
const errorMessage = ref("");
const hasSearched = ref(false);

const results = reactive<MobileSearchResults>({
	query: "",
	tracks: [],
	albums: [],
	artists: [],
	playlists: [],
});

const hasResults = computed(() => {
	return (
		results.tracks.length ||
		results.albums.length ||
		results.artists.length ||
		results.playlists.length
	);
});

async function handleSearch() {
	const term = searchTerm.value.trim();
	if (!term) return;

	isSearching.value = true;
	errorMessage.value = "";

	try {
		const response = await fetchData("mainSearch", { term });
		results.query = response.QUERY || term;

		results.tracks = (response.TRACK?.data || [])
			.map((track) => formatSingleTrack(track))
			.slice(0, 6);
		results.albums = (response.ALBUM?.data || [])
			.map((album) => formatAlbums(album))
			.slice(0, 4);
		results.artists = (response.ARTIST?.data || [])
			.map((artist) => formatArtist(artist))
			.slice(0, 6);
		results.playlists = (response.PLAYLIST?.data || [])
			.map((playlist) => formatPlaylist(playlist))
			.slice(0, 4);
	} catch (error) {
		console.error(error);
		errorMessage.value = "La recherche est momentanément indisponible.";
	} finally {
		isSearching.value = false;
		hasSearched.value = true;
	}
}

function queueLink(link?: string) {
	if (!link) return;
	sendAddToQueue(link);
}
</script>

<template>
	<section class="mobile-search">
		<header class="mobile-search__header">
			<h1>Rechercher</h1>
			<p>Ajoutez rapidement des morceaux à votre file depuis votre téléphone.</p>
		</header>

		<form class="mobile-search__form" @submit.prevent="handleSearch">
			<input
				v-model.trim="searchTerm"
				type="search"
				name="mobile-search"
				placeholder="Artiste, album ou lien Deezer…"
				required
			/>
			<button type="submit" :disabled="isSearching">
				{{ isSearching ? "Recherche…" : "Rechercher" }}
			</button>
		</form>

		<p v-if="errorMessage" class="mobile-search__error">{{ errorMessage }}</p>

		<section v-if="isSearching" class="mobile-card mobile-card--notice">
			Chargement des résultats…
		</section>

		<section
			v-else-if="hasSearched && !hasResults"
			class="mobile-card mobile-card--notice"
		>
			Aucun résultat trouvé pour « {{ results.query }} ».
		</section>

		<template v-else-if="hasResults">
			<section class="mobile-card mobile-card--list">
				<h2>Morceaux</h2>
				<ul class="mobile-search__tracks">
					<li v-for="track in results.tracks" :key="track.trackLink">
						<div>
							<p class="mobile-search__title">{{ track.trackTitle }}</p>
							<p class="mobile-search__subtitle">{{ track.artistName }}</p>
						</div>
						<button
							type="button"
							class="mobile-chip"
							@click="queueLink(track.trackLink)"
						>
							Ajouter
						</button>
					</li>
				</ul>
			</section>

			<section class="mobile-card mobile-card--list">
				<h2>Artistes</h2>
				<div class="mobile-search__grid">
					<RouterLink
						v-for="artist in results.artists"
						:key="artist.artistID"
						class="mobile-search__card-link"
						:to="{ name: 'MobileArtistDetail', params: { id: artist.artistID } }"
					>
						<article class="mobile-search__card">
							<p class="mobile-search__title">{{ artist.artistName }}</p>
							<p class="mobile-search__subtitle">
								{{ artist.artistAlbumsNumber }} sorties
							</p>
							<button
								type="button"
								class="mobile-chip mobile-chip--full"
								@click.stop="queueLink(artist.artistLink)"
							>
								Ajouter l'artiste
							</button>
						</article>
					</RouterLink>
				</div>
			</section>

			<section class="mobile-card mobile-card--list">
				<h2>Albums & playlists</h2>
				<div class="mobile-search__grid">
					<RouterLink
						v-for="album in results.albums"
						:key="album.albumID"
						class="mobile-search__card-link"
						:to="{ name: 'MobileAlbumDetail', params: { id: album.albumID } }"
					>
						<article class="mobile-search__card">
							<p class="mobile-search__title">{{ album.albumTitle }}</p>
							<p class="mobile-search__subtitle">{{ album.artistName }}</p>
							<button
								type="button"
								class="mobile-chip mobile-chip--full"
								@click.stop="queueLink(album.albumLink)"
							>
								Ajouter l'album
							</button>
						</article>
					</RouterLink>

					<article
						v-for="playlist in results.playlists"
						:key="playlist.playlistID"
						class="mobile-search__card"
					>
						<p class="mobile-search__title">{{ playlist.playlistTitle }}</p>
						<p class="mobile-search__subtitle">
							{{ playlist.artistName }}
						</p>
						<button
							type="button"
							class="mobile-chip mobile-chip--full"
							@click="queueLink(playlist.playlistLink)"
						>
							Ajouter la playlist
						</button>
					</article>
				</div>
			</section>
		</template>

		<section v-else class="mobile-card mobile-card--notice">
			Lancez une recherche pour afficher les résultats.
		</section>
	</section>
</template>

<style scoped>
.mobile-search {
	display: flex;
	flex-direction: column;
	gap: 1rem;
	padding-bottom: 4rem;
}

.mobile-search__header h1 {
	font-size: 1.6rem;
	margin-bottom: 0.3rem;
}

.mobile-search__header p {
	font-size: 0.95rem;
	opacity: 0.8;
}

.mobile-search__form {
	display: flex;
	gap: 0.6rem;
	flex-wrap: wrap;
}

.mobile-search__form input {
	flex: 1 1 220px;
	border-radius: 14px;
	border: 1px solid rgba(255, 255, 255, 0.2);
	background: rgba(0, 0, 0, 0.3);
	padding: 0.85rem 1rem;
	color: inherit;
}

.mobile-search__form button {
	border: none;
	border-radius: 14px;
	background: var(--accent-primary, #a46cf5);
	color: #0f0f0f;
	padding: 0.85rem 1.25rem;
	font-weight: 600;
}

.mobile-search__error {
	color: #ff9b9b;
}

.mobile-search__tracks {
	display: flex;
	flex-direction: column;
	gap: 0.75rem;
}

.mobile-search__tracks li {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 0.75rem;
}

.mobile-search__title {
	font-weight: 600;
}

.mobile-search__subtitle {
	font-size: 0.85rem;
	opacity: 0.75;
}

.mobile-search__grid {
	display: grid;
	grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
	gap: 0.75rem;
}

.mobile-search__card {
	border: 1px solid rgba(255, 255, 255, 0.15);
	border-radius: 16px;
	padding: 0.85rem;
	background: rgba(255, 255, 255, 0.03);
}

.mobile-search__card-link {
	color: inherit;
	text-decoration: none;
}

.mobile-chip {
	border: 1px solid rgba(255, 255, 255, 0.25);
	background: none;
	color: inherit;
	border-radius: 999px;
	padding: 0.35rem 0.9rem;
	font-size: 0.85rem;
}

.mobile-chip--full {
	width: 100%;
	text-align: center;
	margin-top: 0.65rem;
}
</style>

