<script setup lang="ts">
import CoverContainer from "@/components/globals/CoverContainer.vue";
import { getHomeData } from "@/data/home";
import { pinia } from "@/stores";
import { useLoginStore } from "@/stores/login";
import { sendAddToQueue } from "@/utils/downloads";
import { setDesktopOverride } from "@/utils/utils";
import { computed, onMounted, ref } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const loginStore = useLoginStore(pinia);

const playlists = ref<any[]>([]);
const albums = ref<any[]>([]);
const isLoadingHome = ref(true);
const linkToQueue = ref("");

const isLoggedIn = computed(() => loginStore.isLoggedIn);

async function bootstrapHome() {
	const homeData = await getHomeData();
	const {
		playlists: { data: playlistData },
		albums: { data: albumData },
	} = homeData;

	playlists.value = playlistData;
	albums.value = albumData;
	isLoadingHome.value = false;
}

function addToQueue(link?: string) {
	const targetLink = link || linkToQueue.value;
	if (!targetLink) return;

	sendAddToQueue(targetLink);
	linkToQueue.value = "";
}

function openDesktopExperience() {
	setDesktopOverride(true);
	const redirectTarget =
		typeof router.currentRoute.value.query.redirect === "string"
			? decodeURIComponent(router.currentRoute.value.query.redirect)
			: "/";
	router.replace({ path: redirectTarget || "/", query: { desktop: "1" } });
}

onMounted(() => {
	bootstrapHome();
});
</script>

<template>
	<section class="mobile-home">
		<div class="mobile-card mobile-card--hero">
			<p class="mobile-eyebrow">Votre musique partout</p>
			<h1>Interface mobile de deemix</h1>
			<p class="mobile-hero__subtitle">
				Gérez vos téléchargements, lancez des recherches rapides et
				ajoutez des titres à la volée, directement depuis votre téléphone.
			</p>

			<button class="mobile-link" type="button" @click="openDesktopExperience">
				Besoin de plus d'options ? Ouvrir la version bureau
			</button>
		</div>

		<section id="mobile-quick-links" class="mobile-card mobile-card--links">
			<h2>Accès rapide</h2>
			<p class="mobile-section__subtitle">
				Ouvrez les sections dédiées pour la recherche mobile ou la gestion des
				téléchargements.
			</p>
			<div class="mobile-quick-links">
				<RouterLink class="mobile-quick-link" :to="{ name: 'MobileSearch' }">
					<i class="material-icons">search</i>
					Recherche mobile
				</RouterLink>
				<RouterLink
					class="mobile-quick-link"
					:to="{ name: 'MobileDownloads' }"
				>
					<i class="material-icons">file_download</i>
					Téléchargements
				</RouterLink>
			</div>
		</section>

		<section
			v-if="!isLoggedIn"
			id="mobile-login-hint"
			class="mobile-card mobile-card--notice"
		>
			<p>
				Connectez-vous depuis la version bureau pour retrouver vos favoris et
				vos paramètres.
			</p>
		</section>

		<section id="mobile-quick-actions" class="mobile-card mobile-card--form">
			<h2>Ajouter un lien</h2>
			<p class="mobile-section__subtitle">
				Collez un lien Deezer ou Spotify pour l'ajouter immédiatement à la
				file d'attente.
			</p>

			<form @submit.prevent="addToQueue" class="mobile-form">
				<input
					v-model.trim="linkToQueue"
					type="url"
					name="deezer-link"
					placeholder="https://www.deezer.com/track/3135556"
					required
				/>
				<button type="submit">Ajouter</button>
			</form>
		</section>

		<section v-if="isLoadingHome" class="mobile-card mobile-card--notice">
			Chargement des recommandations…
		</section>

		<section
			v-else-if="playlists.length"
			id="mobile-playlists"
			class="mobile-card mobile-card--list"
		>
			<div class="mobile-card__header">
				<h2>Playlists populaires</h2>
				<span>{{ playlists.length }} propositions</span>
			</div>
			<div class="mobile-scroll-row">
				<article
					v-for="release in playlists"
					:key="release.id"
					class="mobile-release"
				>
					<CoverContainer
						is-rounded
						:cover="release.picture_medium"
						:link="release.link"
						@click.stop="addToQueue(release.link)"
					/>
					<p class="primary-text">{{ release.title }}</p>
					<p class="secondary-text">{{ release.user.name }}</p>
					<button type="button" class="mobile-release__action" @click="addToQueue(release.link)">
						Ajouter à la file
					</button>
				</article>
			</div>
		</section>

		<section
			v-if="!isLoadingHome && albums.length"
			id="mobile-albums"
			class="mobile-card mobile-card--list"
		>
			<div class="mobile-card__header">
				<h2>Albums populaires</h2>
				<span>{{ albums.length }} propositions</span>
			</div>
			<div class="mobile-scroll-row">
				<article
					v-for="release in albums"
					:key="release.id"
					class="mobile-release"
				>
					<CoverContainer
						is-rounded
						:cover="release.cover_medium"
						:link="release.link"
						@click.stop="addToQueue(release.link)"
					/>
					<p class="primary-text">{{ release.title }}</p>
					<p class="secondary-text">{{ release.artist.name }}</p>
					<button type="button" class="mobile-release__action" @click="addToQueue(release.link)">
						Ajouter à la file
					</button>
				</article>
			</div>
		</section>

		<section class="mobile-card mobile-card--notice">
			<p>
				L'interface mobile se concentre sur les actions rapides. Pour accéder à
				l'ensemble des paramètres et parcourir vos bibliothèques, ouvrez la
				version bureau.
			</p>
		</section>
	</section>
</template>

<style scoped>
.mobile-home {
	display: flex;
	flex-direction: column;
	gap: 1rem;
	padding-bottom: 5rem;
}

.mobile-card {
	background: rgba(255, 255, 255, 0.03);
	border: 1px solid rgba(255, 255, 255, 0.08);
	border-radius: 20px;
	padding: 1.25rem;
	box-shadow: 0 10px 25px rgba(0, 0, 0, 0.25);
}

.mobile-card--hero {
	background: linear-gradient(
		145deg,
		rgba(28, 28, 28, 0.95),
		rgba(86, 54, 149, 0.85)
	);
	text-align: left;
	color: #fff;
}

.mobile-eyebrow {
	text-transform: uppercase;
	font-size: 0.75rem;
	letter-spacing: 0.2rem;
	opacity: 0.8;
	margin-bottom: 0.6rem;
}

.mobile-card--hero h1 {
	font-size: 1.8rem;
	margin-bottom: 0.5rem;
}

.mobile-hero__subtitle {
	opacity: 0.85;
	margin-bottom: 1rem;
	line-height: 1.4;
}

.mobile-link {
	color: inherit;
	text-decoration: underline;
	text-underline-offset: 4px;
	border: none;
	background: none;
	padding: 0;
	font-weight: 600;
}

.mobile-card--notice {
	border-style: dashed;
	border-color: rgba(255, 255, 255, 0.15);
}

.mobile-card--form form {
	display: flex;
	flex-direction: column;
	gap: 0.75rem;
	margin-top: 0.75rem;
}

.mobile-card--form input {
	border-radius: 12px;
	border: 1px solid rgba(255, 255, 255, 0.15);
	background: rgba(0, 0, 0, 0.2);
	padding: 0.75rem 1rem;
	color: inherit;
}

.mobile-card--form button[type="submit"] {
	border: none;
	border-radius: 12px;
	background: var(--accent-primary, #a46cf5);
	color: #0f0f0f;
	padding: 0.8rem 1rem;
	font-weight: 600;
}

.mobile-card__header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	margin-bottom: 0.75rem;
}

.mobile-card--links {
	gap: 0.75rem;
	display: flex;
	flex-direction: column;
}

.mobile-quick-links {
	display: grid;
	grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
	gap: 0.75rem;
}

.mobile-quick-link {
	display: flex;
	flex-direction: column;
	align-items: flex-start;
	gap: 0.35rem;
	border: 1px solid rgba(255, 255, 255, 0.2);
	border-radius: 16px;
	padding: 0.9rem;
	background: rgba(255, 255, 255, 0.04);
	color: inherit;
}

.mobile-quick-link i {
	font-size: 1.2rem;
}

.mobile-scroll-row {
	display: flex;
	gap: 1rem;
	overflow-x: auto;
	padding-bottom: 0.5rem;
}

.mobile-release {
	min-width: 160px;
	display: flex;
	flex-direction: column;
	gap: 0.4rem;
}

.mobile-release__action {
	margin-top: auto;
	border: 1px solid rgba(255, 255, 255, 0.2);
	background: none;
	color: inherit;
	border-radius: 999px;
	padding: 0.35rem 0.75rem;
	font-size: 0.85rem;
}

.primary-text {
	font-weight: 600;
}

.secondary-text {
	font-size: 0.85rem;
	opacity: 0.7;
}
</style>

