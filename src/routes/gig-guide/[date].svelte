<script context="module">
	import API from '$lib/contentful/';
	import { formatDate, groupBy, formatDay, createCalendarLink } from '$lib/globals.mjs';
	import SeoSocial from '$lib/components/seo-social.svelte';
	import PlaylistPromo from '$lib/components/playlist.svelte';

	let gigCounter = 0;
	let whichPrompt = 0;
	const increment = () => { gigCounter++; return ""; }
	const resetCounter = () => { if (gigCounter > 9) gigCounter = 0; return ""; }
	const incrementDisplay = () => { whichPrompt++; return ""; }

	const getGigs = async (date) => {
    let d;
    // check we have something that at least resembles YYYMMDD
    if ((typeof date === 'string') && (date.length === 8) && (parseInt(date) == date)) {
      d = new Date(date.slice(0,4), date.slice(4, 6)-1, date.slice(6,8));
    } else {
      d = new Date();
    }

		const data = await API(`{
      eventsCollection(
        order: gigStartDate_ASC,
        limit: 1000,
        where: { gigStartDate_gte: "${new Date(d.setHours(0)).toISOString()}" }
      ) {
        items {
          gigStartDate
          promotedName
          ticketUrl
          performersList
          furtherInfo
          furtherInfoContributorInitials
		  isFree
          venue {
            venueName
            address
            suburb
            url
          }
        }
      }
    }`);

		if (data) {
			let event = data.eventsCollection.items.map((i) => {
				let { gigStartDate, ...rest } = i;
				let d = new Date(gigStartDate);
        let dateString = `${d.getFullYear()}${`00${d.getMonth()+1}`.slice(-2)}${`00${d.getDate()}`.slice(-2)}`;
				return {
					date: d,
          dateString: dateString,
					time:(d.getHours() % 12) + ":" + d.getMinutes().toString().padStart(2, "0") + (d.getHours() >= 12 ? "pm" : "am"),
					...rest
				};
			});

			let byMonth = groupBy(event, (i) => formatDate(i.date));

			// Group by month
			return byMonth.map((month) => {
				return {
					...month,
					items: groupBy(month.items, (i) => `${i.date.getDate()}:${formatDay(i.date)}`)
				};
			});
		}
		return {};
	};

	export async function load({ url, params }) {
    const date = params && params.date;
		let gigs = await getGigs(date);

		return {
			props: {
				gigs,
        url,
        params
			}
		};
	}
</script>

<script>
	import Event from '$lib/components/event.svelte';
	import Button from '$lib/components/button.svelte';
	import Feedprompt from '../../lib/components/feedprompt.svelte';
	export let gigs;

	let gigcount = 0;
</script>

<SeoSocial title="Gig Guide" />

<picture>
	<source
		srcset="/canman-gigs@2x.png 2560w, /canman-gigs@1x.png 1280w"
		media="(min-width : 640px)" />
	<source
		srcset="/canman-gigs-mobile.png"
		media="(max-width : 640px)" />
	<img
		src="/canman-gigs@1x.png"
		alt="SydneyMusic.net mascot Can Man loves a gig"
		class="aspect-3/1 sm:aspect-banner object-cover w-full mx-auto lg:max-w-5xl"
		 />
</picture>

<div class="max-w-5xl px-5 mt-10 mx-auto space-y-4 md:hidden">
	<h1 class="notch-left text-xl">Gig Guide</h1>

	<div class="prose prose-sm px-3">
		<p>Below you'll find every live music event taking place in Sydney, all on one page.</p>
	</div>

	<div class="transition">
        <div class="accordion-header cursor-pointer transition flex items-center">
			<h3 class="mb-0 notch-left text-lg lg:text-xl">
				How Does This Work?
			</h3>
		</div>
		<div class="accordion-content overflow-hidden max-h-0">
			<div class="prose prose-sm">
				<p>
					This guide is as simple as we can practically get away with. We’ll include some
					occasional commentary (feel free to submit your own!) to help give you context on
					what can be a dizzyingly complex network of musicians, collectives, communities,
					and spaces, or just make sure you don’t miss out on catching your next favourite
					act.
				</p>
				<p>
					Got a gig you think should be listed here?<br />&raquo; <a href="mailto:contact@sydneymusic.net">Drop us an email to submit a show</a>.
				</p>

				<p>Want to read up on how this works?<br />&raquo; <a href="/about">Head over to our About / FAQs page</a>.</p>
				<p>
					<span class="font-bold text-sm">Artists, managers, promoters, and venues:</span><br/>
					Self-promo is fine — we love it when you let us know what you’ve got going on! But we won’t
					publish your marketing/social copy verbatim or give you special consideration
					in the guide. We generally don’t list cover/tribute bands or background-music sets at hospitality
					venues. All listings are at our own discretion. We will also graciously refuse any offer
					of door spots for shows where we can buy tickets.
				</p>
			</div>
		</div>
	</div>
	<div class="transition">
		<div class="accordion-header cursor-pointer transition flex items-center">
			<h3 class="mb-0 notch-left text-lg lg:text-xl">
				Gig Guide Playlist
			</h3>
		</div>
		<div class="accordion-content overflow-hidden max-h-0">
			<PlaylistPromo />
		</div>
	</div>
</div>

<div class="max-w-5xl px-5 mt-10 mx-auto space-y-32 pb-24">
	<!-- First section -->
	<div class="space-y-10">
		<div class="grid md:grid-cols-sidebar-right-wide">
			<!-- left col -->
			<div class="space-y-10 sm:pr-20">
				<div class="filterbox">
					<label for="toggle-freegigs" class="flex items-center cursor-pointer relative mb-4">
						<input type="checkbox" id="toggle-freegigs" class="sr-only">
						<div class="toggle-bg bg-gray-200 border-2 border-gray-200 h-6 w-11 rounded-full"></div>
						<span class="ml-2">FREE GIGS ONLY</span>
					  </label>
				</div>

				{#each gigs as month}
					<div class="guide-month space-y-10">
						<h3 class="notch-left text-lg lg:text-xl">
							{month.label}
						</h3>
						<div class="grid space-y-10">
							{#each month.items as { label, items }}
								<div class="relative day flex items-start">
									<div
										class="sticky -top-5 grid text-center items-center justify-center pr-8 sm:pl-3 sm:pr-10 font-bold"
									>
										<a href={`/gig-guide/${items[0].dateString}`} class="no-underline hover:underline" >
											<p class="text-ruby font-semibold text-base sm:text-lg leading-none uppercase">
												{label.split(':')[1]}
											</p>
											<p class="text-3xl sm:text-4xl leading-none">{label.split(':')[0]}</p>
										</a>
									</div>
									<div class="w-full space-y-5">
										{#each items as event, index}
											<Event
												name={event.promotedName}
												performers={event.performersList}
												calendarLink={createCalendarLink(event)}
												venue={event.venue}
												website={event.ticketUrl}
												comment={event.furtherInfo}
												initials={event.furtherInfoContributorInitials}
												time={event.time}
												isFree={event.isFree}
                        dateString={event.dateString}
                        id={index}
											/>
											{increment()}
										{/each}
									</div>
								</div>
								{#if gigCounter > 9}
								<Feedprompt Index={whichPrompt} />
								{resetCounter() }
								{incrementDisplay()}
								{/if}
							{/each}
						</div>
					</div>
				{/each}
			</div>
			<!-- right col -->
			<div class="space-y-10 mt-20 md:mt-0">
				<h3 class="notch-left text-lg lg:text-xl"><span class="text-ruby">NEW!</span> Support Us!</h3>
				<div class="prose prose-sm">
					<a href="https://store.sydneymusic.net"><img src="/store-promo.jpg" alt="SydneyMusic Store Items: Tote, Tee, Stubby Holder" /></a>
					<p>We're an officially registered not-for-profit these days, and we now have an <a href="https://store.sydneymusic.net">online store</a> where you can make donations and/or buy merch.</p>
				</div>

				<PlaylistPromo showtitle="true" />

				<h3 class="notch-left text-lg lg:text-xl">about the guide</h3>
				<div class="prose prose-sm">
					<p>
						This guide is as simple as we can practically get away with. We’ll include some
						occasional commentary (feel free to submit your own!) to help give you context on
						what can be a dizzyingly complex network of musicians, collectives, communities,
						and spaces, or just make sure you don’t miss out on catching your next favourite
						act.
					</p>
					<p>
						Got a gig you think should be listed here? <a href="mailto:contact@sydneymusic.net"
							>Drop us an email</a
						>.
					</p>
				</div>
				<div class="space-y-3">
					<Button label="Submit a gig" href="mailto:contact@sydneymusic.net" />
				</div>

				<div class="prose prose-sm">
				<p>
					<span class="font-bold text-sm">Artists, managers, promoters, and venues:</span><br/>
					Self-promo is fine —we love it when you let us know what you’ve got going on! But we won’t
					publish your marketing/social copy verbatim or give you special consideration
					in the guide. We generally don’t list cover/tribute bands or background-music sets at hospitality
					venues. All listings are at our own discretion. We will also graciously refuse any offer
					of door spots for shows where we can buy tickets.
				</p>
				</div>
				<div class="space-y-3">
					<Button label="Join the Discord!" href="https://discord.gg/jv8VKrXymJ" />
					<Button label="Other links" href="/links" />
				</div>
			</div>
		</div>
	</div>
</div>


<style>
    .accordion-content {
    transition: max-height 0.3s ease-out, padding 0.3s ease;
    }
</style>
