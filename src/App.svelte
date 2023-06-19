<script>
  import Beeswarm from "./Beeswarm.svelte";
  import Scatterplot from "./Scatterplot.svelte";
  import { onMount } from "svelte"
	import { csv } from "d3-fetch"

  const clubs = [
    {
      league: 'English Premier League',
      clubsToHighlight: ['Manchester City', 'Manchester United', 'Arsenal', 'Chelsea', 'Liverpool'],
      colorRange: ["#00FFFF", "#FF0000", "#FFD700", "#0000FF", "#008000", "#dcdcdc"]
    },
    {
      league: 'Spain Primera Division',
      clubsToHighlight: ['Real Madrid CF', 'FC Barcelona', 'Atlético de Madrid'],
      colorRange: ["#FFD700", "#FF0000", "#0000FF", "#dcdcdc"]
    },
    {
      league: 'Italian Serie A',
      clubsToHighlight: ['Inter', 'Juventus', 'Napoli'],
      colorRange: ["#0000FF", "#000000", "#00FFFF", "#dcdcdc"]
    },
    {
      league: 'German 1. Bundesliga',
      clubsToHighlight: ['FC Bayern München', 'Borussia Dortmund'],
      colorRange: ["#0066B2", "#FDE100", "#dcdcdc"]
    }
  ]

	let dataset = []
 	onMount(
		async () => {
		  dataset = await csv('../src/data/players_22.csv')
        .then((data) => {
          data = data.filter(d => d['league_level'] == "1")
          //data = data.filter(d => d['league_name'] == 'French Ligue 1' || d['league_name'] == 'German 1. Bundesliga' || d['league_name'] == 'Spain Primera Division' || d['league_name'] == 'English Premier League' || d['league_name'] == 'Italian Serie A')
          data.forEach(d => {
            d['overall'] = +d['overall']
            d['wage_eur'] = +d['wage_eur']
            //d['club_name'] = d['club_name'].replace(' ', '_')
          })
          return data;
        });
    }
	)

</script>
<main>
  <h1>FIFA 22 - Official Football Game from EA SPORTS</h1>
  {#if dataset.length > 0}
  <h2>Spain Primera Division Club Comparison</h2>
  <div class='charts-wrapper'>
    <div class='scatter'>
      <Scatterplot 
        data={dataset.filter(d => d['league_name'] == 'Spain Primera Division')}
        clubs={clubs.find(d => d['league'] == 'Spain Primera Division')}
      />
    </div>
    <div class='beeswarm'>
      <Beeswarm 
        data={dataset.filter(d => d['league_name'] == 'Spain Primera Division')}
        clubs={clubs.find(d => d['league'] == 'Spain Primera Division')}
      />
    </div>
  </div>
  <h2>Italian Serie A Club Comparison</h2>
  <div class='charts-wrapper'>
    <div class='scatter'>
      <Scatterplot 
        data={dataset.filter(d => d['league_name'] == 'Italian Serie A')}
        clubs={clubs.find(d => d['league'] == 'Italian Serie A')}
      />
    </div>
    <div class='beeswarm'>
      <Beeswarm 
        data={dataset.filter(d => d['league_name'] == 'Italian Serie A')}
        clubs={clubs.find(d => d['league'] == 'Italian Serie A')}
      />
    </div>
  </div>
  <h2>English Premiere League Club Comparison</h2>
  <div class='charts-wrapper'>
    <div class='scatter'>
      <Scatterplot 
        data={dataset.filter(d => d['league_name'] == 'English Premier League')}
        clubs={clubs.find(d => d['league'] == 'English Premier League')}
      />
    </div>
    <div class='beeswarm'>
      <Beeswarm 
        data={dataset.filter(d => d['league_name'] == 'English Premier League')}
        clubs={clubs.find(d => d['league'] == 'English Premier League')}
      />
    </div>
  </div>
  <h2>German 1. Bundesliga Club Comparison</h2>
  <div class='charts-wrapper'>
    <div class='scatter'>
      <Scatterplot 
        data={dataset.filter(d => d['league_name'] == 'German 1. Bundesliga')}
        clubs={clubs.find(d => d['league'] == 'German 1. Bundesliga')}
      />
    </div>
    <div class='beeswarm'>
      <Beeswarm 
        data={dataset.filter(d => d['league_name'] == 'German 1. Bundesliga')}
        clubs={clubs.find(d => d['league'] == 'German 1. Bundesliga')}
      />
    </div>
  </div>
  {/if}
</main>

<style>
  main {
    text-align: center;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    background: #f0f0f0;
    width: 100vw;
    height: auto;
  }

  .charts-wrapper {
    display: flex;
    flex-direction: row;
    width: 100%
  }

  .beeswarm {
    width: 65%;
    padding: 40px;
  }

  .scatter {
    width: 35%;
    padding: 40px;
  }

  h1 {
    font-size: 2rem;
    margin-bottom: 1rem;
    font-weight: 700;
  }

  h2 {
    font-size: 1.5rem;
    color: #333;
    margin-bottom: 2rem;
    line-height: 1.5;
  }
</style>
