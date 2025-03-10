<script context="module">
	import API from '$lib/contentful/';
	import { formatDate, groupBy, formatDay, createCalendarLink } from '$lib/globals.mjs';
	import SeoSocial from '$lib/components/seo-social.svelte';

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
        order: gigStartDate_DESC,
        limit: 1000,
        where: { gigStartDate_lte: "${new Date(d.setHours(24)).toISOString()}" }
      ) {
        items {
          gigStartDate
          promotedName
          ticketUrl
          performersList
          furtherInfo
          furtherInfoContributorInitials
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
	export let gigs;
</script>

<SeoSocial title="Gig Guide Archive" />

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

<div class="max-w-5xl px-5 mt-10 mx-auto space-y-4">
	<h1 class="notch-left text-xl">Gig Guide <span class="text-ruby">Archive</span></h1>
    <div class="px-3">
        <p>"We have to go back, Marty!!!"</p>
        <p>Yes, this is time travel.</p>
    </div>
</div>

<div class="max-w-5xl px-5 mt-10 mx-auto space-y-32 pb-24">
	<!-- First section -->
	<div class="space-y-10">
		<div class="grid md:grid-cols-sidebar-right-wide">
			<!-- left col -->
			<div class="space-y-10 sm:pr-20">
				{#each gigs as month}
					<div class="space-y-10">
						<h3 class="notch-left text-lg lg:text-xl">
							{month.label}
						</h3>
						<div class="grid space-y-10">
							{#each month.items as { label, items }}
								<div class="relative day flex items-start">
									<div
										class="sticky top-5 grid text-center items-center justify-center pr-8 sm:pl-3 sm:pr-10 font-bold"
									>
                    <a href={`/gig-guide/archive/${items[0].dateString}`} class="no-underline hover:underline" >
                      <p class="text-ruby font-semibold text-base sm:text-lg leading-none uppercase">
                        {label.split(':')[1]}
                      </p>
                      <p class="text-3xl sm:text-4xl leading-none">{label.split(':')[0]}</p>
                      <p class="text-neutral-500 font-normal text-base sm:text-xs leading-none uppercase pt-1">{month.label}</p>
                    </a>
									</div>
									<div class="space-y-5 w-full">
										{#each items as event, index}
											<div>
												<Event
													name={event.promotedName}
													performers={event.performersList}
													calendarLink={createCalendarLink(event)}
													venue={event.venue}
													website={event.ticketUrl}
													comment={event.furtherInfo}
                          initials={event.furtherInfoContributorInitials}
													time={event.time}
													dateString={event.dateString}
													id={index}
												/>
											</div>
										{/each}
									</div>
								</div>
							{/each}
						</div>
					</div>
				{/each}
			</div>
			<!-- right col -->
			<div class="space-y-10 mt-20 md:mt-0">

				<h3 class="notch-left text-lg lg:text-xl">about the archive</h3>
				<div class="prose prose-sm">
					<p>
						This is just what's in the database. It may have errors.
					</p>

                    <p>We're interested in the notion of creating a more complete archive of gigs that have taken place in Sydney over the years... but we're not there just yet. For now, enjoy this!</p>
				</div>
			</div>
		</div>
	</div>
</div>
