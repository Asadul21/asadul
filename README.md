<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Pokédex</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      margin: 0;
      font-family: sans-serif;
      overflow-x: hidden;
      background: linear-gradient(
        to bottom,
        #ef5350 0%,
        #ef5350 45%,
        #000000 45%,
        #000000 48%,
        #ffffff 48%,
        #ffffff 100%
      );
      background-attachment: fixed;
      position: relative;
    }

    /* Floating Pokéballs */
    .pokeball {
      position: absolute;
      width: 40px;
      height: 40px;
      background: url('https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/items/poke-ball.png') no-repeat center/contain;
      animation: float 20s linear infinite;
      opacity: 0.4;
      z-index: 0;
    }
    @keyframes float {
      0% { transform: translateY(100vh) translateX(0); }
      100% { transform: translateY(-150vh) translateX(50px); }
    }
    .pokeball:nth-child(1) { left: 5%; animation-duration: 18s; }
    .pokeball:nth-child(2) { left: 20%; animation-duration: 22s; }
    .pokeball:nth-child(3) { left: 40%; animation-duration: 19s; }
    .pokeball:nth-child(4) { left: 60%; animation-duration: 24s; }
    .pokeball:nth-child(5) { left: 80%; animation-duration: 21s; }

    /* Header base style */
    header {
      background: #ffffff;
      text-align: center;
      padding: 20px;
      color: #3b4cca;
      font-size: 32px;
      font-weight: bold;
      position: relative;
      z-index: 1;
      animation: neonPulse 2.5s ease-in-out infinite;
    }
    @keyframes neonPulse {
      0%, 100% {
        color: #3b4cca;
        text-shadow: 0 0 5px #3b4cca, 0 0 10px #3b4cca, 0 0 20px #3b4cca;
      }
      50% {
        color: #d62828;
        text-shadow: 0 0 15px #d62828, 0 0 30px #d62828, 0 0 40px #d62828;
      }
    }

    /* Center search and buttons */
    .search-box {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      margin: 0 auto;
      width: 100%;
      padding: 10px 0 0 0;
    }
    #search {
      width: 320px;
      padding: 10px 16px;
      border-radius: 8px;
      border: 1px solid #ccc;
      margin-bottom: 14px;
      font-size: 1rem;
      box-sizing: border-box;
      text-align: center;
      display: block;
      background: #fff;
    }
    #region-buttons {
      text-align: center;
      padding: 0 0 10px 0;
      width: 100%;
    }
    #region-buttons button {
      margin: 5px;
      padding: 6px 12px;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-weight: bold;
      transition: all 0.3s ease;
    }
    #region-buttons button:hover {
      filter: brightness(85%);
      transform: scale(1.05);
    }
    #region-buttons button.active {
      box-shadow: 0 0 12px #fff;
      transform: scale(1.1);
    }
    /* Distinct background colors for regions */
    #region-buttons button[data-region="all"] {
      background-color: #3b4cca; /* Blue */
    }
    #region-buttons button[data-region="kanto"] {
      background-color: #FF0000; /* Red */
    }
    #region-buttons button[data-region="johto"] {
      background-color: #4CAF50; /* Green */
    }
    #region-buttons button[data-region="hoenn"] {
      background-color: #FF9800; /* Orange */
    }
    #region-buttons button[data-region="sinnoh"] {
      background-color: #9C27B0; /* Purple */
    }
    #region-buttons button[data-region="unova"] {
      background-color: #00BCD4; /* Cyan */
    }
    #region-buttons button[data-region="kalos"] {
      background-color: #8BC34A; /* Light Green */
    }
    #region-buttons button[data-region="alola"] {
      background-color: #795548; /* Brown */
    }
    #region-buttons button[data-region="galar"] {
      background-color: #E91E63; /* Pink */
    }
    #region-buttons button[data-region="mega"] {
      background-color: #607D8B; /* Blue Gray */
    }

    /* Container, cards and rest of styles unchanged */
    .container {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 16px;
      padding: 20px;
      position: relative;
      z-index: 1;
    }
    .card {
      background: white;
      border-radius: 12px;
      width: 160px;
      padding: 10px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      text-align: center;
      cursor: pointer;
      transition: transform .3s;
    }
    .card:hover {
      transform: scale(1.05);
      box-shadow: 0 6px 16px rgba(0,0,0,0.2);
    }
    @keyframes landing {
      0% {
        transform: scale(1) translateY(0);
        box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      }
      50% {
        transform: scale(1.1) translateY(-20px);
        box-shadow: 0 8px 20px rgba(0,0,0,0.3);
      }
      100% {
        transform: scale(1) translateY(0);
        box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      }
    }
    .card.landing {
      animation: landing 0.6s ease forwards;
    }
    .card img {
      width: 100%;
      height: 120px;
      object-fit: contain;
    }
    .modal {
      position: fixed;
      top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(0,0,0,0.6);
      display: flex; justify-content: center; align-items: center;
      visibility: hidden; opacity: 0;
      transition: visibility 0.3s, opacity 0.3s;
      z-index: 10;
    }
    .modal.active {
      visibility: visible;
      opacity: 1;
    }
    .modal-content {
      background: white;
      border-radius: 12px;
      padding: 20px;
      max-width: 90vw;
      width: 95%;
      max-height: 90vh;
      overflow-y: auto;
      box-shadow: 0 0 20px black;
      position: relative;
    }
    .modal-content h2 {
      margin-top: 0;
    }
    .evolution {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 20px;
    }
    .evolution-item {
      text-align: center;
      cursor: pointer;
      border: 1px solid #ccc;
      border-radius: 8px;
      padding: 10px;
      background: #f9f9f9;
      width: 150px;
      transition: transform 0.3s, box-shadow 0.3s;
    }
    .evolution-item:hover {
      transform: scale(1.08);
      box-shadow: 0 0 12px #3b4cca;
    }
    .evolution-item img {
      width: 100px;
    }
    .close-btn {
      position: absolute;
      top: 10px;
      right: 10px;
      background: red;
      color: white;
      border: none;
      padding: 5px 10px;
      cursor: pointer;
    }
    .nav-buttons {
      text-align: center;
      margin-top: 15px;
    }
    .nav-buttons button {
      padding: 8px 16px;
      margin: 0 10px;
      background: #3b4cca;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }
    .nav-buttons button:hover {
      background: #2a379f;
    }
    canvas {
      max-width: 100%;
      margin-top: 20px;
    }
  </style>
</head>
<body>

  <!-- Floating Pokéballs -->
  <div class="pokeball"></div>
  <div class="pokeball"></div>
  <div class="pokeball"></div>
  <div class="pokeball"></div>
  <div class="pokeball"></div>

  <header>Pokédex</header>

  <div class="search-box">
    <input type="text" id="search" placeholder="Search Pokémon...">
    <div id="region-buttons">
      <button data-region="all" onclick="filterByRegion('all')" class="active">All</button>
      <button data-region="kanto" onclick="filterByRegion('kanto')">Kanto</button>
      <button data-region="johto" onclick="filterByRegion('johto')">Johto</button>
      <button data-region="hoenn" onclick="filterByRegion('hoenn')">Hoenn</button>
      <button data-region="sinnoh" onclick="filterByRegion('sinnoh')">Sinnoh</button>
      <button data-region="unova" onclick="filterByRegion('unova')">Unova</button>
      <button data-region="kalos" onclick="filterByRegion('kalos')">Kalos</button>
      <button data-region="alola" onclick="filterByRegion('alola')">Alola</button>
      <button data-region="galar" onclick="filterByRegion('galar')">Galar</button>
      <button data-region="mega" onclick="filterByRegion('mega')">Mega</button>
    </div>
  </div>

  <div class="container" id="pokedex">Loading Pokémon...</div>

  <div class="modal" id="modal">
    <div class="modal-content">
      <button class="close-btn" onclick="closeModal()">X</button>
      <div id="modal-body">Loading...</div>
      <div class="nav-buttons">
        <button onclick="navigatePokemon(-1)">⬅ Previous</button>
        <button onclick="navigatePokemon(1)">Next ➡</button>
      </div>
    </div>
  </div>

  <script>
    const API_BASE = 'https://pokeapi.co/api/v2';
    let allPokemons = [], filteredPokemons = [], currentIndex = 0;
    let chartInstance = null;
    let megaPokemons = null; // cache for mega list

    async function fetchAllPokemons() {
      const res = await fetch(`${API_BASE}/pokemon?limit=1000`);
      const data = await res.json();
      const results = data.results;

      allPokemons = await Promise.all(results.map(async (p) => {
        const details = await fetch(p.url).then(r => r.json());
        return {
          id: details.id,
          name: details.name,
          img: details.sprites.other['official-artwork'].front_default
        };
      }));

      filteredPokemons = [...allPokemons];
      renderPokemons(filteredPokemons);
    }

    function renderPokemons(list) {
      const container = document.getElementById('pokedex');
      container.innerHTML = '';
      list.forEach((p, i) => {
        const card = document.createElement('div');
        card.className = 'card';
        card.innerHTML = `<img src="${p.img}" alt="${p.name}"><div>${formatMegaName(p.name)}</div>`;

        card.onclick = () => {
          card.classList.remove('landing');
          void card.offsetWidth;
          card.classList.add('landing');

          setTimeout(() => {
            currentIndex = i;
            speak(p.name);
            showPokemonDetails(p.name);
          }, 600);
        };

        container.appendChild(card);
      });
    }

    async function showPokemonDetails(name) {
      try {
        const res = await fetch(`${API_BASE}/pokemon/${name}`);
        const pokemon = await res.json();

        const speciesRes = await fetch(pokemon.species.url);
        const species = await speciesRes.json();

        const evoRes = await fetch(species.evolution_chain.url);
        const evoData = await evoRes.json();
        const evoNames = getEvoNames(evoData.chain);

        const evoDetails = await Promise.all(
          evoNames.map(async n => {
            try {
              return await fetch(`${API_BASE}/pokemon/${n}`).then(r => r.json());
            } catch {
              return null;
            }
          })
        );

        const modal = document.getElementById('modal');
        const body = document.getElementById('modal-body');
        modal.classList.add('active');
        body.innerHTML = `
          <h2>${formatMegaName(pokemon.name)} (#${pokemon.id})</h2>
          <img src="${pokemon.sprites.other['official-artwork'].front_default}" width="150">
          <p><b>Height:</b> ${pokemon.height / 10} m</p>
          <p><b>Weight:</b> ${pokemon.weight / 10} kg</p>
          <p><b>Types:</b> ${pokemon.types.map(t => capitalize(t.type.name)).join(', ')}</p>
          <p><b>Abilities:</b> ${pokemon.abilities.map(a => capitalize(a.ability.name)).join(', ')}</p>
          <h3>Stats:</h3>
          <canvas id="statsChart"></canvas>
          <h3>Evolution Chain:</h3>
          <div class="evolution">
            ${evoDetails.filter(Boolean).map(evo => `
              <div class="evolution-item" onclick="handleEvoClick('${evo.name}')">
                <img src="${evo.sprites.other['official-artwork'].front_default}">
                <div style="${evo.name === pokemon.name ? 'font-weight:bold;color:#d62828' : ''}">
                  ${formatMegaName(evo.name)}
                </div>
                <small>HP: ${evo.stats[0].base_stat} | ATK: ${evo.stats[1].base_stat}</small>
              </div>
            `).join('')}
          </div>
        `;
        drawChart(pokemon.stats);
      } catch (err) {
        document.getElementById('modal-body').innerHTML = "Error loading data.";
        console.error(err);
      }
    }

    function drawChart(stats) {
      const ctx = document.getElementById('statsChart').getContext('2d');
      if (chartInstance) chartInstance.destroy();
      chartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: stats.map(s => capitalize(s.stat.name)),
          datasets: [{
            label: 'Base Stat',
            data: stats.map(s => s.base_stat),
            backgroundColor: '#3b4cca'
          }]
        },
        options: {
          responsive: true,
          scales: { y: { beginAtZero: true } }
        }
      });
    }

    function getEvoNames(chain) {
      const result = [];
      function recurse(node) {
        result.push(node.species.name);
        node.evolves_to.forEach(recurse);
      }
      recurse(chain);
      return result;
    }

    function closeModal() {
      document.getElementById('modal').classList.remove('active');
      if (chartInstance) chartInstance.destroy();
    }

    function speak(text) {
      speechSynthesis.cancel();
      const utter = new SpeechSynthesisUtterance(formatMegaName(text));
      utter.lang = 'en-US';
      speechSynthesis.speak(utter);
    }

    function handleEvoClick(name) {
      const idx = filteredPokemons.findIndex(p => p.name === name);
      if (idx !== -1) currentIndex = idx;
      speak(name);
      showPokemonDetails(name);
    }

    function capitalize(str) {
      return str.charAt(0).toUpperCase() + str.slice(1);
    }

    function formatMegaName(str) {
      if (!str) return "";
      return str
        .replace(/-/g, ' ')
        .replace(/\bmega\b/i, 'Mega')
        .replace(/\b(x|y)\b/gi, m => m.toUpperCase())
        .replace(/^(.)|\s+(.)/g, c => c.toUpperCase());
    }

    function navigatePokemon(dir) {
      currentIndex = (currentIndex + dir + filteredPokemons.length) % filteredPokemons.length;
      const p = filteredPokemons[currentIndex];
      speak(p.name);
      showPokemonDetails(p.name);
    }

    async function filterByRegion(region) {
      const regions = {
        all: [1, 1010],
        kanto: [1, 151],
        johto: [152, 251],
        hoenn: [252, 386],
        sinnoh: [387, 493],
        unova: [494, 649],
        kalos: [650, 721],
        alola: [722, 809],
        galar: [810, 905],
      };

      // Remove active class from all buttons
      const buttons = document.querySelectorAll('#region-buttons button');
      buttons.forEach(btn => btn.classList.remove('active'));

      // Add active class to clicked button
      event.target.classList.add('active');

      if (region === 'mega') {
        if (!megaPokemons) {
          const res = await fetch(`${API_BASE}/pokemon?limit=2000`);
          const data = await res.json();
          const megaResults = data.results.filter(p => p.name.toLowerCase().includes("mega"));
          megaPokemons = await Promise.all(megaResults.map(async (p) => {
            try {
              const details = await fetch(p.url).then(r => r.json());
              return {
                id: details.id,
                name: details.name,
                img: details.sprites.other['official-artwork'].front_default
              };
            } catch {
              return null;
            }
          }));
          megaPokemons = megaPokemons.filter(Boolean);
        }
        filteredPokemons = [...megaPokemons];
        renderPokemons(filteredPokemons);
        return;
      }

      const [min, max] = regions[region];
      filteredPokemons = allPokemons.filter(p => p.id >= min && p.id <= max);
      renderPokemons(filteredPokemons);
    }

    document.getElementById('search').addEventListener('input', e => {
      const q = e.target.value.toLowerCase();
      filteredPokemons = allPokemons.filter(p => p.name.includes(q));
      renderPokemons(filteredPokemons);
    });

    document.addEventListener('keydown', e => {
      if (document.getElementById('modal').classList.contains('active')) {
        if (e.key === 'ArrowLeft') navigatePokemon(-1);
        if (e.key === 'ArrowRight') navigatePokemon(1);
        if (e.key === 'Escape') closeModal();
      }
    });

    fetchAllPokemons();
  </script>
</body>
</html>
