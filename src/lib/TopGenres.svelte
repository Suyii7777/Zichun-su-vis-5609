<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";

  type Props = {
    movies: TMovie[];
  };

  let { movies }: Props = $props();

  // Find the top 3 genres for every year
  function getTopGenres(movies: TMovie[]) {
    const yearGenreCounts = new Map<number, Map<string, number>>();

    movies.forEach((movie) => {
      const year = movie.year.getFullYear();

      if (!yearGenreCounts.has(year)) {
        yearGenreCounts.set(year, new Map());
      }

      const genreCounts = yearGenreCounts.get(year)!;

      movie.genres.forEach((genre) => {
        genreCounts.set(genre, (genreCounts.get(genre) || 0) + 1);
      });
    });

    const result: {
      year: number;
      rank: number;
      genre: string;
      count: number;
    }[] = [];

    yearGenreCounts.forEach((genreCounts, year) => {
      const top3 = Array.from(genreCounts.entries())
        .sort((a, b) => b[1] - a[1])
        .slice(0, 3);

      top3.forEach(([genre, count], index) => {
        result.push({
          year,
          rank: index + 1,
          genre,
          count
        });
      });
    });

    return result.sort((a, b) => a.year - b.year || a.rank - b.rank);
  }

  const topGenres = $derived(getTopGenres(movies));

  const width = 850;
  const height = 260;

  const margin = {
    top: 25,
    right: 30,
    bottom: 50,
    left: 70
  };

  const genres = $derived(
    Array.from(new Set(topGenres.map((d) => d.genre)))
  );

  const years = $derived(topGenres.map((d) => d.year));

  const minYear = $derived(
    years.length > 0 ? d3.min(years) ?? 1946 : 1946
  );

  const maxYear = $derived(
    years.length > 0 ? d3.max(years) ?? 2024 : 2024
  );

  const xScale = $derived(
    d3
      .scaleLinear()
      .domain([minYear, maxYear + 1])
      .range([margin.left, width - margin.right])
  );

  const yScale = d3
    .scaleBand<number>()
    .domain([1, 2, 3])
    .range([margin.top, height - margin.bottom])
    .padding(0.15);

  const genreColors: Record<string, string> = {
  Drama: "#4daf4a",
  Comedy: "#4db6ac",
  Romance: "#e15759",
  Action: "#f1c40f",
  Crime: "#9c6ade",
  Thriller: "#76b7b2",
  Adventure: "#4e79a7",
  Horror: "#f28e2b",
  Fantasy: "#edc948",
  Family: "#b6992d",
  Mystery: "#af7aa1",
  Biography: "#b07aa1",
  Animation: "#ff9da7",
  "Sci-Fi": "#59a14f",
  War: "#9c755f",
  Western: "#8cd17d",
  Sport: "#e7969c",
  "Film-Noir": "#bab0ab"
};

function colorScale(genre: string) {
  return genreColors[genre] ?? "#bdbdbd";
}
</script>

<h2>Annual Top 3 Genres Over Time</h2>

<p class="description">
  Each rectangle represents one genre in the annual top three.
</p>

<svg {width} {height}>
  <!-- Rank labels -->
  {#each [1, 2, 3] as rank}
    <text
      x={margin.left - 12}
      y={(yScale(rank) ?? 0) + yScale.bandwidth() / 2}
      text-anchor="end"
      dominant-baseline="middle"
    >
      Rank {rank}
    </text>
  {/each}

  <!-- Ribbon rectangles -->
  {#each topGenres as d}
    <rect
      x={xScale(d.year)}
      y={yScale(d.rank)}
      width={Math.max(2, xScale(d.year + 1) - xScale(d.year))}
      height={yScale.bandwidth()}
      fill={colorScale(d.genre)}
    >
      <title>
        {d.year} — Rank {d.rank}: {d.genre} ({d.count} movies)
      </title>
    </rect>
  {/each}

  <!-- Year labels -->
  {#each d3.range(
    Math.ceil(minYear / 10) * 10,
    maxYear + 1,
    10
  ) as year}
    <text
      x={xScale(year)}
      y={height - 25}
      text-anchor="middle"
      font-size="11"
    >
      {year}
    </text>

    <line
      x1={xScale(year)}
      x2={xScale(year)}
      y1={height - margin.bottom}
      y2={height - margin.bottom + 5}
      stroke="black"
    />
  {/each}

  <!-- X axis line -->
  <line
    x1={margin.left}
    x2={width - margin.right}
    y1={height - margin.bottom}
    y2={height - margin.bottom}
    stroke="black"
  />

  <text
    x={width / 2}
    y={height - 5}
    text-anchor="middle"
    font-size="12"
  >
    Year
  </text>
</svg>

<div class="legend">
  {#each genres as genre}
    <span class="legend-item">
      <span
        class="legend-color"
        style={`background-color: ${colorScale(genre)}`}
      ></span>
      {genre}
    </span>
  {/each}
</div>

<style>
  svg {
    max-width: 100%;
    height: auto;
  }

  .description {
    margin-bottom: 5px;
  }

  .legend {
    max-width: 850px;
    display: flex;
    flex-wrap: wrap;
    gap: 8px 14px;
    margin-top: 8px;
    font-size: 12px;
  }

  .legend-item {
    display: flex;
    align-items: center;
    gap: 4px;
  }

  .legend-color {
    width: 12px;
    height: 12px;
    display: inline-block;
  }

  rect:hover {
    stroke: black;
    stroke-width: 2px;
  }
</style>