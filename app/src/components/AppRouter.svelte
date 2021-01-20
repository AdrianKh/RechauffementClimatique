<script>
  import Bande from "./Bande.svelte"
  import Metropoles from "./Metropoles.svelte"
  import France from "./France.svelte"
  import Cercles from "./Cercles.svelte"
  import Home from "./Home.svelte"

  const headers = {
    "home":{
      'href': '/'
    },
    "Bande": {
      'href': 'Bande'
    },
    "Metropoles": {
      'href': 'Metropoles' 
    },
    "France": {
      'href': 'France' 
    },
    "Cercles": {
      'href': 'Cercles' 
    }
  }

  let getCurrentRoute = () => window.location.hash;
  let currentHash = getCurrentRoute();
  window.onhashchange = () => currentHash = getCurrentRoute();
  if (getCurrentRoute() == '') window.location.href = '/#/';

  const url = ""

</script>

  <nav class="navbar navbar-expand-lg navbar-light bg-light">
    <div class="navbar-brand">
      <a href="#{headers.home.href}">Home</a>
    </div>
    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>

    
    <div class="collapse navbar-collapse" id="navbarNav">
      <ul class="navbar-nav">
        <li class="nav-item active">
          <div class="nav-link"> 
            <a class="nav-link active" href="#{headers.Bande.href}">Bande</a>
          </div>
        </li>
        <li class="nav-item">
          <div class="nav-link">
            <a class="nav-link active" href="#{headers.Metropoles.href}">Metropoles</a>
          </div>
        </li>
        <li class="nav-item">
          <div class="nav-link">
            <a class="nav-link active" href="#{headers.France.href}">France</a>
          </div>
        </li>
        <li class="nav-item">
          <div class="nav-link">
            <a class="nav-link active" href="#{headers.Cercles.href}">Cercles</a>
          </div>
        </li>
      </ul>
    </div>
  </nav>
  
  {#if currentHash == '#' + headers.home.href}
    <Home></Home>
  {/if}
  {#if currentHash == '#' + headers.Bande.href}
    <Bande></Bande>
  {/if}
  {#if currentHash == '#' + headers.Metropoles.href}
    <Metropoles></Metropoles>
  {/if}
  {#if currentHash == '#' + headers.France.href}
    <France></France>
  {/if}
  {#if currentHash == '#' + headers.Cercles.href}
    <Cercles></Cercles>
  {/if}

