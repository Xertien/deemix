<script setup lang="ts">
import { setDesktopOverride } from "@/utils/utils";
import { computed, onMounted, onUnmounted, ref, watch } from "vue";
import { useRoute, useRouter } from "vue-router";

const router = useRouter();
const route = useRoute();
const isMenuOpen = ref(false);

const menuPages = [
	{ name: "MobileHome", label: "Accueil" },
	{ name: "MobileDownloads", label: "Téléchargements" },
	{ name: "MobileSearch", label: "Recherche" },
];

const navAnchors = [
	{ id: "mobile-home-top", label: "Accueil" },
	{ id: "mobile-quick-links", label: "Accès rapide" },
	{ id: "mobile-quick-actions", label: "Actions" },
	{ id: "mobile-playlists", label: "Playlists" },
	{ id: "mobile-albums", label: "Albums" },
];

const isHomeRoute = computed(() => route.name === "MobileHome");

function toggleMenu() {
	isMenuOpen.value = !isMenuOpen.value;
}

function goToPage(pageName: string) {
	if (route.name === pageName) {
		isMenuOpen.value = false;
		return;
	}

	router.push({ name: pageName });
	isMenuOpen.value = false;
}

function scrollToSection(anchorId: string) {
	if (!isHomeRoute.value) return;

	const section = document.getElementById(anchorId);
	if (section) {
		section.scrollIntoView({ behavior: "smooth", block: "start" });
	}
	isMenuOpen.value = false;
}

function switchToDesktop() {
	setDesktopOverride(true);

	const target =
		typeof route.query.redirect === "string" && route.query.redirect.length
			? decodeURIComponent(route.query.redirect)
			: "/";

	router.replace(target || "/");
}

watch(
	() => route.fullPath,
	() => {
		isMenuOpen.value = false;
	}
);

onMounted(() => {
	if (typeof document === "undefined") return;
	document.body.classList.add("mobile-ui-active");
});

onUnmounted(() => {
	if (typeof document === "undefined") return;
	document.body.classList.remove("mobile-ui-active");
});
</script>

<template>
	<div class="mobile-layout">
		<header class="mobile-layout__header" id="mobile-home-top">
			<button
				class="mobile-layout__menu"
				type="button"
				aria-label="Ouvrir le menu"
				@click="toggleMenu"
			>
				<svg
					class="mobile-layout__menu-icon"
					viewBox="0 0 24 24"
					role="img"
					aria-hidden="true"
				>
					<path
						fill="currentColor"
						d="M3 6h18a1 1 0 0 0 0-2H3a1 1 0 1 0 0 2zm18 5H3a1 1 0 1 0 0 2h18a1 1 0 0 0 0-2zm0 7H3a1 1 0 1 0 0 2h18a1 1 0 0 0 0-2z"
					/>
				</svg>
			</button>

			<div class="mobile-layout__brand">
				<strong>deemix</strong>
				<span>mobile</span>
			</div>

			<button
				class="mobile-layout__desktop"
				type="button"
				aria-label="Passer en mode bureau"
				@click="switchToDesktop"
			>
				Mode bureau
			</button>
		</header>

		<nav v-if="isMenuOpen" class="mobile-layout__nav">
			<button
				v-for="page in menuPages"
				:key="page.name"
				type="button"
				class="mobile-layout__nav-link"
				:class="{ 'mobile-layout__nav-link--active': route.name === page.name }"
				@click="goToPage(page.name)"
			>
				{{ page.label }}
			</button>

			<template v-if="isHomeRoute">
				<button
					v-for="anchor in navAnchors"
					:key="anchor.id"
					type="button"
					class="mobile-layout__nav-link mobile-layout__nav-link--anchor"
					@click="scrollToSection(anchor.id)"
				>
					{{ anchor.label }}
				</button>
			</template>
		</nav>

		<main class="mobile-layout__content">
			<router-view />
		</main>
	</div>
</template>

<style scoped>
:global(body.mobile-ui-active) {
	overflow-y: auto;
	background: var(--main-background, #050505);
	color: var(--text-primary, #f5f5f5);
}

.mobile-layout {
	min-height: 100vh;
	display: flex;
	flex-direction: column;
	background: var(--main-background, #050505);
	color: var(--text-primary, #f5f5f5);
}

.mobile-layout__header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 1rem 1.25rem;
	background: rgba(0, 0, 0, 0.8);
	backdrop-filter: blur(8px);
	position: sticky;
	top: 0;
	z-index: 20;
}

.mobile-layout__brand {
	display: flex;
	align-items: baseline;
	gap: 0.35rem;
	font-size: 1.1rem;
	text-transform: uppercase;
	letter-spacing: 0.15rem;
}

.mobile-layout__brand span {
	font-size: 0.8rem;
	letter-spacing: 0.2rem;
	color: rgba(255, 255, 255, 0.6);
}

.mobile-layout__menu {
	display: flex;
	align-items: center;
	justify-content: center;
	background: none;
	border: 1px solid rgba(255, 255, 255, 0.35);
	border-radius: 999px;
	padding: 0.35rem;
	color: #fff;
}

.mobile-layout__menu-icon {
	width: 22px;
	height: 22px;
}

.mobile-layout__desktop {
	border: 1px solid rgba(255, 255, 255, 0.2);
	background: none;
	color: inherit;
	padding: 0.35rem 0.8rem;
	border-radius: 999px;
	font-size: 0.85rem;
}

.mobile-layout__nav {
	display: flex;
	flex-direction: column;
	gap: 0.5rem;
	padding: 0.75rem 1rem 1.25rem;
	background: rgba(0, 0, 0, 0.85);
}

.mobile-layout__nav-link {
	border: 1px solid rgba(255, 255, 255, 0.2);
	border-radius: 14px;
	padding: 0.8rem 1rem;
	background: rgba(255, 255, 255, 0.05);
	color: inherit;
	text-align: left;
	font-size: 0.95rem;
}

.mobile-layout__nav-link--active {
	background: rgba(255, 255, 255, 0.12);
	border-color: rgba(255, 255, 255, 0.5);
}

.mobile-layout__nav-link--anchor {
	border-style: dashed;
	font-size: 0.85rem;
}

.mobile-layout__content {
	flex: 1 0 auto;
	padding: 1.25rem;
}

@media (min-width: 768px) {
	.mobile-layout__content {
		margin: 0 auto;
		max-width: 540px;
		width: 100%;
	}
}
</style>

