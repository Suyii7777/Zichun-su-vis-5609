<script lang="ts">
    import * as d3 from "d3";
    import { Scatter, Line } from "$lib";
    import RuntimeBar from "$lib/RuntimeBar.svelte";
    import { onMount } from "svelte";
    import type { TMovie } from "../../types";

    // Reactive variable for storing the data
    let movies: TMovie[] = $state([]);

    let yearRange: [Date, Date] | undefined = $state();

    function getYearCountArray(movies: TMovie[]) {
        let yearCount: { [year: number]: number } = {};
        const allYears = [...new Set(movies.map((d) => d.year.getFullYear()))];
        for (let year of allYears) {
            yearCount[year] = movies.filter(
                (d) => d.year.getFullYear() == year,
            ).length;
        }

        // Convert the map to an array of { year, count } objects
        const yearCountArray = Object.entries(yearCount).map(
            ([year, count]) => ({
                x: new Date(year),
                y: count as number,
            }),
        );

        // Sort the array by year in ascending order
        yearCountArray.sort((a, b) => (a.x < b.x ? -1 : 1));
        return yearCountArray;
    }

    let yearCountArray = $derived(getYearCountArray(movies));

function getYearRuntimeArray(movies: TMovie[]) {
    const allYears = [...new Set(movies.map((d) => d.year.getFullYear()))];

    const yearRuntimeArray = allYears.map((year) => {
        const moviesInYear = movies.filter(
            (d) => d.year.getFullYear() === year &&
                d.runtime_minutes > 0
        );

        const totalRuntime = moviesInYear.reduce(
            (sum, d) => sum + d.runtime_minutes, 0
        );

        return {
            x: new Date(year, 0, 1),
            y: moviesInYear.length > 0
                ? totalRuntime / moviesInYear.length
                : 0,
        };
    });

    yearRuntimeArray.sort((a, b) => a.x.getTime() - b.x.getTime());
    return yearRuntimeArray;
}

let yearRuntimeArray = $derived(getYearRuntimeArray(movies));
let runtimeYearRange: [Date, Date] | undefined = $state();

    type TAxisSelection = {
        x: keyof TMovie;
        y: keyof TMovie;
        size: keyof TMovie;
    };

    // tip4: axisSelection is a reactive variable that stores the axis selection
    let axisSelection: TAxisSelection = $state({
        x: "year",
        y: "average_rating",
        size: "num_votes",
    });

    // Function to load the CSV
    async function loadCsv() {
        try {
            const csvUrl = "./summer_movies.csv";
            movies = await d3.csv(csvUrl, (row) => {
                // all values are strings, so use row conversion function to format them
                return {
                    ...row,
                    num_votes: Number(row.num_votes),
                    runtime_minutes: Number(row.runtime_minutes),
                    genres: row.genres.split(","),
                    year: new Date(row.year),
                    average_rating: Number(row.average_rating),
                };
            });

            console.log("Loaded CSV Data:", movies);
        } catch (error) {
            console.error("Error loading CSV:", error);
        }
    }
    // Call the loader when the component mounts
    onMount(loadCsv);

    // tip4: below are the derived stores for the options of the selectors
    const attrOptionsX = $derived(movies[0] ? Object.keys(movies[0]) : []);
    const attrOptionsY = $derived(
        movies[0] ? Object.keys(movies[0]).filter((d) => d != "genres") : [],
    );
    const attrOptionsS = $derived(
        movies[0] ? Object.keys(movies[0]).filter((d) => d != "genres") : [],
    );
</script>

<div class="container">
    <h1>Summer Movies</h1>

    <p>Here are {movies.length == 0 ? "..." : movies.length + " "} movies</p>
    {#if movies.length > 0}
       <div class="selectors">
    <label>
        X Axis:
        <select bind:value={axisSelection.x}>
            {#each attrOptionsX as key}
                <option value={key}>{key}</option>
            {/each}
        </select>
    </label>

    <label>
        Y Axis:
        <select bind:value={axisSelection.y}>
            {#each attrOptionsY as key}
                <option value={key}>{key}</option>
            {/each}
        </select>
    </label>

    <label>
        Size:
        <select bind:value={axisSelection.size}>
            {#each attrOptionsS as key}
                <option value={key}>{key}</option>
            {/each}
        </select>
    </label>
</div>
        <Scatter
            movies={yearRange
                ? movies.filter(
                      (d) => d.year <= yearRange[1] && d.year >= yearRange[0],
                  )
                : movies}
            x={axisSelection.x}
            y={axisSelection.y}
            size={axisSelection.size}
        />
        <br />

        <Line data={yearCountArray} bind:yearRange />
    {/if}
 
{#if movies.length > 0}
    <hr />

    <h2>Part 4: Summer Movie Runtime Analysis</h2>
    <p>How has the runtime of summer movies changed over time?</p>

    <h3>Movie Runtime by Year</h3>

    <Scatter
        movies={runtimeYearRange
            ? movies.filter(
                (d) =>
                    d.year >= runtimeYearRange[0] &&
                    d.year <= runtimeYearRange[1]
              )
            : movies}
        x="year"
        y="runtime_minutes"
        size="num_votes"
    />

    <h3>Average Movie Runtime by Year</h3>

    <RuntimeBar
        data={yearRuntimeArray}
        bind:yearRange={runtimeYearRange}
    />
{/if}
   
</div>

<style>
    .container {
        width: 60vw;
        margin: 10px auto;
        padding: 10px;
    }
</style>
