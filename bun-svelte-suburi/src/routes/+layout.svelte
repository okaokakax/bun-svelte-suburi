<!-- ふわっと遷移させるコード -->

<script>
	import { onNavigate } from '$app/navigation';

	onNavigate((navigation) => {
		if (!document.startViewTransition) return;

		return new Promise((resolve) => {
			document.startViewTransition(async () => {
				resolve();
				await navigation.complete;
			});
		});
	});

	import { Container, Navbar, NavbarBrand, Nav, NavItem, NavLink } from 'sveltestrap';
</script>

<Navbar color="light" light>
	<NavbarBrand href="/">Bun + Svelte 素振り</NavbarBrand>
	<Nav>
		<NavItem>
			<NavLink href="/geolocation-test">GeoLocation APIの素振り</NavLink>
		</NavItem>
	</Nav>
</Navbar>

<Container>
	<slot />
</Container>
