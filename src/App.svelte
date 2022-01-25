<script>
  import { onMount } from "svelte";
  import SvelteTable from "svelte-table";

  const statsUrl = "https://everrise.azurewebsites.net/stats";
  const refreshRate = 50;

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
      renderValue: (v) => parseInt(v.holders),
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
      renderValue: (v) => parseInt(v.holders),
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

    if (unifiedRows.length != 0) {
      const numUpdates = 50;
      const timeBetweenUpdates = 100;
      const old = unifiedRows[0];
      const priceIncr = (getPrice("unified") - old.price) / numUpdates;
      const marketCapIncr =
        (getMarketCap("unified") - old.marketCap) / numUpdates;
      const holdersIncr = (getHolders("unified") - old.holders) / numUpdates;
      const stakedIncr = (getStaked("unified") - old.staked) / numUpdates;

      let incrs = [];

      rows.forEach((row) => {
        incrs.push({
          price: (getPrice(row.id) - row.price) / numUpdates,
          marketCap: (getMarketCap(row.id) - row.marketCap) / numUpdates,
          holders: (getHolders(row.id) - row.holders) / numUpdates,
          staked: (getStaked(row.id) - row.staked) / numUpdates,
        });
      });

      for (let i = 0; i < numUpdates; i++) {
        const old = unifiedRows[0];

        unifiedRows[0] = {
          chain: "ALL UNIFIED",
          price: old.price + priceIncr,
          marketCap: old.marketCap + marketCapIncr,
          holders: old.holders + holdersIncr,
          staked: old.staked + stakedIncr,
        };

        unifiedRows = unifiedRows;

        rows.forEach((row, i) => {
          rows[i] = {
            id: row.id,
            chain: row.chain,
            price: row.price + incrs[i].price,
            marketCap: row.marketCap + incrs[i].marketCap,
            holders: row.holders + incrs[i].holders,
            staked: row.staked + incrs[i].staked,
          };
          rows = rows;
        });

        await new Promise((resolve, reject) => {
          setTimeout(() => resolve(1), timeBetweenUpdates);
        });
      }
    } else {
      // UNIFIED
      unifiedRows.push({
        id: "unified",
        chain: "ALL UNIFIED",
        price: getPrice("unified"),
        marketCap: getMarketCap("unified"),
        holders: getHolders("unified"),
        staked: getStaked("unified"),
      });

      // BSC
      rows.push({
        id: "bsc",
        chain: "Binance",
        price: getPrice("bsc"),
        marketCap: getMarketCap("bsc"),
        holders: getHolders("bsc"),
        staked: getStaked("bsc"),
      });

      // Polygon
      rows.push({
        id: "poly",
        chain: "Polygon",
        price: getPrice("poly"),
        marketCap: getMarketCap("poly"),
        holders: getHolders("poly"),
        staked: getStaked("poly"),
      });

      // Ethreum
      rows.push({
        id: "eth",
        chain: "Ethreum",
        price: getPrice("eth"),
        marketCap: getMarketCap("eth"),
        holders: getHolders("eth"),
        staked: getStaked("eth"),
      });

      // Fantom
      rows.push({
        id: "ftm",
        chain: "Fantom",
        price: getPrice("ftm"),
        marketCap: getMarketCap("ftm"),
        holders: getHolders("ftm"),
        staked: getStaked("ftm"),
      });

      // Avalanche
      rows.push({
        id: "avax",
        chain: "Avalanche",
        price: getPrice("avax"),
        marketCap: getMarketCap("avax"),
        holders: getHolders("avax"),
        staked: getStaked("avax"),
      });

      unifiedRows = unifiedRows;
      rows = rows;
    }

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
<details class:hidden={isLoading}>
  <summary>Details</summary>
  <pre>{formattedData}</pre>
</details>

<style>
  .hidden {
    visibility: hidden;
  }
</style>
