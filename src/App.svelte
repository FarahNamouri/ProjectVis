<script>
  import { Router, Route, Link } from "svelte-routing";
  import PieChart from "./components/PieChart.svelte";
  import AccPerTerritory from "./components/AccPerTerritory.svelte";
  import OrdersPerTerr from "./components/OrdersPerTerr.svelte";
</script>

  <style>
  main {
    background: radial-gradient(circle, rgba(255, 223, 223, 1) 0%, rgba(203, 232, 255, 1) 100%);
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 1em;
    margin: 0 auto;
    max-width: 1200px;
  }

  nav {
    width: 100%;
    background-color: #f0f0f0;
    padding: 10px;
    margin-bottom: 20px; 
  }

  nav div {
    cursor: pointer;
    text-align: center;
    display: inline-block;
    padding: 10px;
    margin: 5px;
    background-color: #e0e0e0;
    border-radius: 5px;
  }

  nav div:hover {
    background-color: #d0d0d0;
  } 
</style> 

<Router>
  <main>
    <h1>D.E.A.D. Dashboard</h1>
    <nav>
      <Link to="/managers"><div>Managers</div></Link>
      <Link to="/company"><div>Company</div></Link>
      <Link to="/orders"><div>Orders</div></Link>
    </nav>

    <Route path="/managers" component={PieChart} />
    <Route path="/company" component={AccPerTerritory} />
    <Route path="/orders" component={OrdersPerTerr} />
  </main>
</Router> 