<script>
	import dayjs from "dayjs";
	import isoWeek from "dayjs/plugin/isoWeek";
	import weekOfYear from "dayjs/plugin/weekOfYear";
import { empty } from "svelte/internal";

	import Modal from "../lib/modal.svelte";
	let modal;

	import Search from "../lib/search.svelte";

	//props
	export let useSearch = true;
	export let useRemote = true;
	export let onlyWeekView = false;
	export let onlyMonthView = false;

	dayjs.extend(isoWeek);
	dayjs.extend(weekOfYear);
	let DayJS = dayjs();

	let numOfRows = 6;
	let numOfCols = 7;
	let maxDays = DayJS.daysInMonth();
	let firstDay = DayJS.date(1);
	let dayPadding = firstDay.format("d");

	let dayMap = {};
	populateMap();

	let chosenView = true;

	let refresh = true;

	let refreshCal = true; //necessary to refresh chechboxes when fetch retrieve the calendars
	let calendars = {};
	populateCalendars();

	function changeCalendarStatus(calName) {
		calendars[calName].used = !calendars[calName].used;
		refresh = !refresh;
	}

	export function getColorFromCalendar(calName) {
		return calendars[calName].color;
	}

	function getCalendarsNames() {
		let tmpList = [];
		let ind = 0;
		for (var c in calendars) {
			tmpList[ind] = calendars[c];
			ind++;
		}
		return tmpList;
	}

	function getCalendars() {
		if (calendars == undefined) return [];
		return getCalendarsNames();
	}

	export function getCalendarMap() {
		return calendars;
	}

	async function getAllCalendars() {
		const res = await fetch(
			`https://supsi-events.herokuapp.com/bff/calendars`
		);
		const json = await res.text();

		if (res.ok) {
			return JSON.parse(json);
		} else {
			// throw new Error("Errore pull calendari"); //dead endpoint, returning empty list
			empty_list = {};
			return empty_list;
		}
	}

	function populateCalendars() {
		let map = {};
		let prom = getAllCalendars();

		return prom.then((cals) => {
			for (let i = 0; i < cals.length; i++) {
				let cal = cals[i];
				map[cal.name] = {
					name: cal.name,
					color: cal.color,
					used: true,
				};
			}
			calendars = map;
			refreshCal = !refreshCal;
		});
	}

	let promise = getAllEvents();
	async function getAllEvents() {
		const res = await fetch(
			`https://supsi-events.herokuapp.com/bff/events`
		);
		const json = await res.text();

		if (res.ok) {
			return JSON.parse(json);
		} else {
			// throw new Error("Errore pull eventi"); //dead endpoint, returning empty list
			empty_list = {};
			return empty_list;
		}
	}

	let dayofweek = dayjs().isoWeekday(6);
	function getDayOfWeek() {
		dayofweek = dayofweek.add(1, "day");
		return dayofweek.format("dddd");
	}

	let cal_title = setCalTitle()
	function setCalTitle() {
		if (onlyWeekView) {
			return "Week " +
				DayJS.week() +
				" (" +
				DayJS.day(0).format("D MMM") +
				" - " +
				DayJS.day(6).format("D MMM") +
				")";
		} else {
			return DayJS.format("MMMM") + " " + DayJS.format("YYYY");
		}
	}

	function previos() {
		if (chosenView) {
			DayJS = DayJS.subtract(1, "month");
			cal_title = DayJS.format("MMMM") + " " + DayJS.format("YYYY");
			firstDay = DayJS.date(1);
			dayPadding = firstDay.format("d");
			maxDays = DayJS.daysInMonth();
		} else {
			DayJS = DayJS.startOf("w");
			DayJS = DayJS.subtract(1, "week");
			cal_title =
				"Week " +
				DayJS.week() +
				" (" +
				DayJS.day(0).format("D MMM") +
				" - " +
				DayJS.day(6).format("D MMM") +
				")";
		}
		refresh = !refresh;
	}

	function today() {
		DayJS = dayjs();
		if (chosenView) {
			cal_title = DayJS.format("MMMM") + " " + DayJS.format("YYYY");
			firstDay = DayJS.date(1);
			dayPadding = firstDay.format("d");
			maxDays = DayJS.daysInMonth();
		} else {
			cal_title =
				"Week " +
				DayJS.week() +
				" (" +
				DayJS.day(0).format("D MMM") +
				" - " +
				DayJS.day(6).format("D MMM") +
				")";
		}
		refresh = !refresh;
	}

	function next() {
		if (chosenView) {
			DayJS = DayJS.add(1, "month");
			cal_title = DayJS.format("MMMM") + " " + DayJS.format("YYYY");
			firstDay = DayJS.date(1);
			dayPadding = firstDay.format("d");
			maxDays = DayJS.daysInMonth();
		} else {
			DayJS = DayJS.startOf("w");
			DayJS = DayJS.add(1, "week");
			cal_title =
				"Week " +
				DayJS.week() +
				" (" +
				DayJS.day(0).format("D MMM") +
				" - " +
				DayJS.day(6).format("D MMM") +
				")";
		}
		refresh = !refresh;
	}

	function changeView() {
		chosenView = !chosenView;
		if (chosenView) {
			document.getElementById("btn-view").innerHTML = "Week View";
			cal_title = DayJS.format("MMMM") + " " + DayJS.format("YYYY");
			numOfRows = 6;
		} else {
			document.getElementById("btn-view").innerHTML = "Month View";
			cal_title =
				"Week " +
				DayJS.week() +
				" (" +
				DayJS.day(0).format("D MMM") +
				" - " +
				DayJS.day(6).format("D MMM") +
				")";
			numOfRows = 1;
		}
		refresh = !refresh;
	}

	function populateMap() {
		let map = {};
		let prom = getAllEvents();

		return prom.then((eventList) => {
			for (let i = 0; i < eventList.length; i++) {
				let event = eventList[i];
				if (map[event.date]) {
					map[event.date].push(event);
				} else {
					map[event.date] = [];
					map[event.date].push(event);
				}
			}
			dayMap = map;
			refresh = !refresh;
		});
	}

	function filterList(list) {
		let tmpList = [];
		for (let i = 0; i < list.length; i++) {
			if (calendars[list[i].calendar].used) {
				tmpList.push(list[i]);
			}
		}
		tmpList.sort((a, b) => {
			return (
				Number(a.timeStart.split(":")[0]) -
				Number(b.timeStart.split(":")[0])
			);
		});
		return tmpList;
	}

	function getEventList(day, monthAndYear) {
		let actualDate = (day < 10 ? "0" + day : day) + "." + monthAndYear;

		let list = dayMap[actualDate];
		if (list == undefined) return [];
		return filterList(list);
	}

	function compareToday(day) {
		if (
			day == dayjs().format("D") &&
			DayJS.format("MM-YYYY") == dayjs().format("MM-YYYY")
		)
			return true;
		return false;
	}

	let i = 1;
	let daysCnt = 0;
	function getDayList() {
		const days = [];
		if (onlyWeekView) {
			chosenView = false; //setting that will use only the week view
		}
		if (chosenView) {
			for (let i = 1; i <= numOfCols * numOfRows; i++) {
				if (daysCnt < maxDays && i > dayPadding) {
					daysCnt++;
					let today = compareToday(daysCnt);
					days.push({
						isActive: true,
						isToday: today,
						number: daysCnt,
						eventList: getEventList(
							daysCnt,
							DayJS.format("MM.YYYY")
						),
					});
				} else {
					days.push({ isActive: false, number: 0, eventList: [] });
				}
			}
			i = 1;
			daysCnt = 0;
		} else {
			let startDay = DayJS.startOf("week");
			for (let i = 1; i <= 7; i++) {
				let num = startDay.format("D");
				let today = compareToday(num);
				days.push({
					isActive: true,
					isToday: today,
					number: num,
					eventList: getEventList(num, startDay.format("MM.YYYY")),
				});
				startDay = startDay.add(1, "day");
			}
		}
		return days;
	}
</script>

<head>
	<title>Calendar component</title>
</head>
<main>
	<div class="topmenu">
		<div class="buttons">
			<button
				class="topmenu-items"
				style="height: 40px; width: 80px;"
				on:click={previos}>Previous</button
			>
			<button
				class="topmenu-items"
				style="height: 40px; width: 80px;"
				on:click={today}>Today</button
			>
			<button
				class="topmenu-items"
				style="height: 40px; width: 80px;"
				on:click={next}>Next</button
			>
		</div>
		<div>
			<h2 class="title" style="font-family: Cambria;">{cal_title}</h2>
		</div>
		<div>
			{#if !onlyWeekView && !onlyMonthView}
				<button
					id="btn-view"
					class="topmenu-items"
					style="height: 40px; width: 100px;"
					on:click={changeView}>Week View</button
				>
			{/if}
		</div>
	</div>
	<div class="grid-container" id="grid-calendar">
		{#each Array(numOfCols) as _, i}
			<div class="grid-days">
				<p style="font-family: Cambria;">{getDayOfWeek(i)}</p>
			</div>
		{/each}
		{#key refresh}
			{#each getDayList() as day}
				{#if day.isToday}
					<div class="grid-today">
						<p style="color: orangered; font-family: Impact;">
							{day.number}
						</p>
						{#each day.eventList as event}
							<div class="event">
								<p
									style="color: {getColorFromCalendar(
										event.calendar
									)};"
								>
									{event.title}
								</p>
								<img
									class="infoImg"
									src="http://cdn.onlinewebfonts.com/svg/img_519962.png"
									alt="info"
									height="12"
									width="12"
									on:click={() =>
										modal.createModalAndShow(event)}
								/>
							</div>
						{/each}
					</div>
				{:else if day.isActive}
					<div class="grid-item">
						<p style="font-family: Impact; color: white;">
							{day.number}
						</p>
						{#each day.eventList as event}
							<div class="event">
								<p
									style="color: {getColorFromCalendar(
										event.calendar
									)};"
								>
									{event.title}
								</p>
								<img
									class="infoImg"
									src="http://cdn.onlinewebfonts.com/svg/img_519962.png"
									alt="info"
									height="12"
									width="12"
									on:click={() =>
										modal.createModalAndShow(event)}
								/>
							</div>
						{/each}
					</div>
				{:else}
					<div class="not-used" />
				{/if}
			{/each}
		{/key}
		<!-- <div
			class="searchResults"
			style="margin-top: 20px; grid-column: 1 / 6;"
		>
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
							<span style="color: {getColorFromCalendar(
								calendar
							)};">{date}</span>
						</li>
						{/each}
					</div>
				{/await}
			</div>
		</div> -->

		{#if useSearch}
			<Search />
		{/if}

		{#key refreshCal}
			<div
				class="checkboxs"
				style="grid-column: 6 / 8; justify-self: center; padding-top: 20px;"
			>
				{#each getCalendars() as cal}
					<label style="color: {cal.color}; padding-right: 20px;">
						{cal.name}
						<input
							type="checkbox"
							checked="true"
							on:click={() => changeCalendarStatus(cal.name)}
						/>
					</label>
				{/each}
			</div>
		{/key}
	</div>

	<Modal bind:this={modal} />
</main>

<style>
	.grid-container {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		background-color: orangered;
		border: 5px solid rgb(0, 0, 0);
		padding: 10px;
	}
	.grid-item {
		background-color: rgb(26, 26, 26);
		border: 1px solid rgba(0, 0, 0, 0.8);
		padding-left: 10px;
		min-height: 40px;
		justify-content: space-between;
	}
	.not-used {
		background-color: rgb(200, 200, 200);
		border: 1px solid rgba(0, 0, 0, 0.9);
	}
	.grid-today {
		background-color: rgb(26, 26, 26);
		border: 2px solid rgb(255, 115, 0);
		padding-left: 10px;
		min-height: 40px;
		justify-content: space-between;
	}
	.grid-days {
		background-color: rgba(255, 255, 255, 0.3);
		border: 1px solid rgba(0, 0, 0, 0.9);
		border-top: 3px solid rgba(0, 0, 0, 0.9);
		border-bottom: 3px solid rgba(0, 0, 0, 0.9);
		padding: 10px;
		font-size: 20px;
		text-align: center;
	}
	.topmenu {
		display: flex;
		padding: 10px;
		justify-content: space-between;
		align-items: center;
		border-top: 0px;
		border: 5px solid rgb(0, 0, 0);
		border-bottom: 0px;
		background-color: orangered;
	}
	.topmenu-items {
		padding: 10px;
		justify-content: space-between;
		border: 2px solid rgb(0, 0, 0);
		background-color: orange;
	}
	.event {
		background-color: rgb(89, 89, 89);
		border: 1px orangered;
		position: relative;
		padding-right: 20px;
	}
	.infoImg {
		position: absolute;
		bottom: 5px;
		right: 5px;
	}
	.infoImg:hover {
		cursor: pointer;
	}
</style>
