<script>
    // import Calendar from "../lib/calendar.svelte";
	// let calendarObj

    let str = "";
	let searchResult = search();
	async function search() {
		if (str != "") {
			const res = await fetch(
				`https://supsi-events.herokuapp.com/bff/events?search=${str}`
			);
			const json = await res.text();

			if (res.ok) {
				return JSON.parse(json);
			} else {
				throw new Error("Errore ricerca degli eventi");
			}
		}
		return [];
	}

	function updateTable() {
		searchResult = search();
	}
</script>

<div class="searchResults" style="margin-top: 20px; grid-column: 1 / 6;">
    <input
        bind:value={str}
        on:change={updateTable}
        placeholder="Search..."
        style="height: 25px; background-color: black; color: white;"
    />
    <div id="search" style="padding-top: 10px;">
        {#await searchResult}
            <p>fetching...</p>
        {:then objs}
            <div>
                {#each objs as { title, date, calendar }}
                    <li>
                        <span>{title}</span>
                        <!-- <span style="color: {calendarObj.getColorFromCalendar(
                            calendar
                        )};">{date}</span> -->
                    </li>
                {/each}
            </div>
        {/await}
    </div>
    <div style="display: none;">
        <!-- <Calendar bind:this={calendarObj}/> -->
    </div>
</div>
