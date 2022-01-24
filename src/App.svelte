<script>
  import { onMount } from "svelte";
  import SvelteTable from "svelte-table";

  const statsUrl = "https://everrise.azurewebsites.net/stats";
  const refreshRate = 3000;

  let isLoading = true;
  const USDFormatter = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    maximumFractionDigits: 0,
  });

  const priceFormatter = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: 8,
  });

  let rawData;
  let formattedData;
  let unifiedRows = [];
  let unifiedColumns = [
    {
      key: "chain",
      title: "BLOCKCHAIN",
      value: (v) => v.chain,
    },
    {
      key: "price",
      title: "PRICE",
      value: (v) => v.price,
      renderValue: (v) => priceFormatter.format(v.price),
    },
    {
      key: "marketCap",
      title: "MKT CAP",
      value: (v) => v.marketCap,
      renderValue: (v) => USDFormatter.format(v.marketCap),
    },
    {
      key: "holders",
      title: "HOLDERS",
      value: (v) => v.holders,
    },
    {
      key: "staked",
      title: "% OF SUPPLY STAKED",
      value: (v) => v.staked,
      renderValue: (v) => v.staked.toFixed(3) + "%",
    },
  ];

  let rows = [];
  let columns = [
    {
      key: "chain",
      title: "BLOCKCHAIN",
      value: (v) => v.chain,
      sortable: true,
    },
    {
      key: "price",
      title: "PRICE",
      value: (v) => v.price,
      renderValue: (v) => priceFormatter.format(v.price),
      sortable: true,
    },
    {
      key: "marketCap",
      title: "MKT CAP",
      value: (v) => v.marketCap,
      renderValue: (v) => USDFormatter.format(v.marketCap),
      sortable: true,
    },
    {
      key: "holders",
      title: "HOLDERS",
      value: (v) => v.holders,
      sortable: true,
    },
    {
      key: "staked",
      title: "% OF SUPPLY STAKED",
      value: (v) => v.staked,
      renderValue: (v) => v.staked.toFixed(3) + "%",
      sortable: true,
    },
  ];

  const getPrice = (symbol) => {
    return parseFloat(rawData[symbol].current.tokenPriceStable, 8);
  };

  const getMarketCap = (symbol) => {
    return parseFloat(rawData[symbol].current.marketCap);
  };

  const getHolders = (symbol) => {
    return parseInt(rawData[symbol].current.holders);
  };

  const getStaked = (symbol) => {
    return parseFloat(rawData[symbol].current.stakedOfTotalSupplyPercent) * 100;
  };

  const updateTables = async () => {
    let res = await fetch(statsUrl);
    rawData = await res.json();

    formattedData = JSON.stringify(rawData, null, 2);

    unifiedRows = [];
    // UNIFIED
    unifiedRows.push({
      chain: "ALL UNIFIED",
      price: getPrice("unified"),
      marketCap: getMarketCap("unified"),
      holders: getHolders("unified"),
      staked: getStaked("unified"),
    });

    rows = [];
    // BSC
    rows.push({
      chain: "Binance",
      price: getPrice("bsc"),
      marketCap: getMarketCap("bsc"),
      holders: getHolders("bsc"),
      staked: getStaked("bsc"),
    });

    // Polygon
    rows.push({
      chain: "Polygon",
      price: getPrice("poly"),
      marketCap: getMarketCap("poly"),
      holders: getHolders("poly"),
      staked: getStaked("poly"),
    });

    // Ethreum
    rows.push({
      chain: "Ethreum",
      price: getPrice("eth"),
      marketCap: getMarketCap("eth"),
      holders: getHolders("eth"),
      staked: getStaked("eth"),
    });

    // Fantom
    rows.push({
      chain: "Fantom",
      price: getPrice("ftm"),
      marketCap: getMarketCap("ftm"),
      holders: getHolders("ftm"),
      staked: getStaked("ftm"),
    });

    // Avalanche
    rows.push({
      chain: "Avalanche",
      price: getPrice("avax"),
      marketCap: getMarketCap("avax"),
      holders: getHolders("avax"),
      staked: getStaked("avax"),
    });

    unifiedRows = unifiedRows;
    rows = rows;

    if (isLoading) {
      isLoading = false;
    }

    setTimeout(updateTables, refreshRate);
  };

  onMount(() => {
    // Format tables
    document.querySelectorAll("table").forEach((table, i) => {
      table.classList.add("table");
      if (i === 0) {
        table.classList.add("table-bordered");
        table.querySelector("tbody").classList.add("text-primary");
      } else {
        table.classList.add("table-striped");
      }
    });

    updateTables();
  });
</script>

<div class="container-fluid text-center">
  {#if isLoading}
    <div class="spinner-border text-warning mt-4" role="status">
      <span class="visually-hidden">Loading...</span>
    </div>
  {/if}
  <div class:hidden={isLoading}>
    <SvelteTable columns={unifiedColumns} rows={unifiedRows} />
    <br />
    <SvelteTable {columns} {rows} />
  </div>
</div>

<br />
<details>
  <summary>Details</summary>
  <pre class:hidden={isLoading}>{formattedData}</pre>
</details>

<style>
  .hidden {
    visibility: hidden;
  }
</style>
