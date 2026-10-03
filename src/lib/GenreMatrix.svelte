<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";
  type Props = {
    movies: TMovie[];
  };

  let { movies }: Props = $props();

  type GenrePair = {
    genre1: string;
    genre2: string;
    count: number;
  };

  function getGenrePairs(movies: TMovie[]): GenrePair[] {
    const pairCounts = new Map<string, number>();

    movies.forEach((movie) => {
      // Remove missing genres and duplicates
      const genres = Array.from(
        new Set(movie.genres.filter((genre) => genre && genre !== "NA"))
      );

      // Create every genre pair within one movie
      for (let i = 0; i < genres.length; i++) {
        for (let j = i + 1; j < genres.length; j++) {
          // Sort so Comedy-Romance and Romance-Comedy
          // are treated as the same pair
          const pair = [genres[i], genres[j]].sort();
          const key = `${pair[0]}|||${pair[1]}`;

          pairCounts.set(key, (pairCounts.get(key) || 0) + 1);
        }
      }
    });

    const result: GenrePair[] = [];

    pairCounts.forEach((count, key) => {
      const [genre1, genre2] = key.split("|||");

      result.push({
        genre1,
        genre2,
        count
      });
    });

    return result.sort((a, b) => b.count - a.count);
  }

  const genrePairs = $derived(getGenrePairs(movies));
  const genres = $derived(
  Array.from(
    new Set(
      genrePairs.flatMap((d) => [d.genre1, d.genre2])
    )
  ).sort()
);

const width = 700;
const height = 700;

const margin = {
  top: 110,
  right: 30,
  bottom: 30,
  left: 110
};

const xScale = $derived(
  d3
    .scaleBand()
    .domain(genres)
    .range([margin.left, width - margin.right])
    .padding(0.1)
);

const yScale = $derived(
  d3
    .scaleBand()
    .domain(genres)
    .range([margin.top, height - margin.bottom])
    .padding(0.1)
);

const maxCount = $derived(
  d3.max(genrePairs, (d) => d.count) ?? 1
);

const sizeScale = $derived(
  d3
    .scaleSqrt()
    .domain([0, maxCount])
    .range([0, 13])
);
</script>

<h2>Genre Co-occurrence Matrix</h2>

<p>Number of genre pairs: {genrePairs.length}</p>

{#if genrePairs.length > 0}
  <p>
    Most frequent pair:
    <strong>
      {genrePairs[0].genre1} + {genrePairs[0].genre2}
    </strong>
    ({genrePairs[0].count} movies)
  </p>
{/if}
{#if genrePairs.length > 0}
  <svg {width} {height}>

    <!-- X-axis genre labels -->
    {#each genres as genre}
      <text
        x={(xScale(genre) ?? 0) + xScale.bandwidth() / 2}
        y={margin.top - 10}
        text-anchor="start"
        font-size="11"
        transform={`rotate(-45 ${(xScale(genre) ?? 0) + xScale.bandwidth() / 2} ${margin.top - 10})`}
      >
        {genre}
      </text>
    {/each}

    <!-- Y-axis genre labels -->
    {#each genres as genre}
      <text
        x={margin.left - 10}
        y={(yScale(genre) ?? 0) + yScale.bandwidth() / 2}
        text-anchor="end"
        dominant-baseline="middle"
        font-size="11"
      >
        {genre}
      </text>
    {/each}

    <!-- Bubbles -->
    {#each genrePairs as pair}
      <circle
        cx={(xScale(pair.genre1) ?? 0) + xScale.bandwidth() / 2}
        cy={(yScale(pair.genre2) ?? 0) + yScale.bandwidth() / 2}
        r={sizeScale(pair.count)}
        fill="#4e79a7"
        opacity="0.75"
      >
        <title>
          {pair.genre1} + {pair.genre2}: {pair.count} movies
        </title>
      </circle>

      <circle
        cx={(xScale(pair.genre2) ?? 0) + xScale.bandwidth() / 2}
        cy={(yScale(pair.genre1) ?? 0) + yScale.bandwidth() / 2}
        r={sizeScale(pair.count)}
        fill="#4e79a7"
        opacity="0.75"
      >
        <title>
          {pair.genre1} + {pair.genre2}: {pair.count} movies
        </title>
      </circle>
    {/each}

  </svg>
{/if}
<style>
  svg {
    max-width: 100%;
    height: auto;
  }

  circle {
    transition: opacity 0.15s ease;
  }

  circle:hover {
    opacity: 1;
    stroke: black;
    stroke-width: 2px;
  }
</style>