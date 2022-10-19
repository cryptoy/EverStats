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
      renderValue: (v) => {
        return `<div class='d-flex justify-content-left align-items-center'>
          <img class='icon me-2' src='${v.icon}'>${v.chain}
          </div>`;
      },
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
      key: "liquidity",
      title: "LIQUIDITY",
      value: (v) => v.liquidity,
      renderValue: (v) => USDFormatter.format(v.liquidity),
    },
    {
      key: "reserves",
      title: "KRAKEN",
      value: (v) => v.reserves,
      renderValue: (v) => USDFormatter.format(v.reserves),
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
      renderValue: (v) => {
        return `<div class='d-flex justify-content-left align-items-center'>
          <img class='icon me-2' src='${v.icon}'>${v.chain}
          </div>`;
      },
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
      key: "liquidity",
      title: "LIQUDITY",
      value: (v) => v.liquidity,
      renderValue: (v) => USDFormatter.format(v.liquidity),
      sortable: true,
    },
    {
      key: "reserves",
      title: "KRAKEN",
      value: (v) => v.reserves,
      renderValue: (v) => USDFormatter.format(v.reserves),
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

  const getLiquidity = (symbol) => {
    return parseFloat(rawData[symbol].current.usdLiquidityCoin);
  };

  const getReserves = (symbol) => {
    return parseFloat(rawData[symbol].current.usdReservesBalance);
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
      const numUpdates = 100;
      const timeBetweenUpdates = 40;
      const old = unifiedRows[0];
      const priceIncr = (getPrice("unified") - old.price) / numUpdates;
      const marketCapIncr =
        (getMarketCap("unified") - old.marketCap) / numUpdates;
      const liquidityIncr =
        (getLiquidity("unified") - old.liquidity) / numUpdates;
      const reservesIncr = (getReserves("unified") - old.reserves) / numUpdates;
      const holdersIncr = (getHolders("unified") - old.holders) / numUpdates;
      const stakedIncr = (getStaked("unified") - old.staked) / numUpdates;

      let incrs = [];

      rows.forEach((row) => {
        incrs.push({
          price: (getPrice(row.id) - row.price) / numUpdates,
          marketCap: (getMarketCap(row.id) - row.marketCap) / numUpdates,
          liquidity: (getLiquidity(row.id) - row.liquidity) / numUpdates,
          reserves: (getReserves(row.id) - row.reserves) / numUpdates,
          holders: (getHolders(row.id) - row.holders) / numUpdates,
          staked: (getStaked(row.id) - row.staked) / numUpdates,
        });
      });

      for (let i = 0; i < numUpdates; i++) {
        const old = unifiedRows[0];

        unifiedRows[0] = {
          id: "unified",
          icon: old.icon,
          chain: "ALL",
          price: old.price + priceIncr,
          marketCap: old.marketCap + marketCapIncr,
          liquidity: old.liquidity + liquidityIncr,
          reserves: old.reserves + reservesIncr,
          holders: old.holders + holdersIncr,
          staked: old.staked + stakedIncr,
        };

        unifiedRows = unifiedRows;

        rows.forEach((row, i) => {
          rows[i] = {
            id: row.id,
            icon: row.icon,
            chain: row.chain,
            price: row.price + incrs[i].price,
            marketCap: row.marketCap + incrs[i].marketCap,
            liquidity: row.liquidity + incrs[i].liquidity,
            reserves: row.reserves + incrs[i].reserves,
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
        icon: "icons/everrise.png",
        chain: "ALL UNIFIED",
        price: getPrice("unified"),
        marketCap: getMarketCap("unified"),
        liquidity: getLiquidity("unified"),
        reserves: getReserves("unified"),
        holders: getHolders("unified"),
        staked: getStaked("unified"),
      });

      // BSC
      rows.push({
        id: "bsc",
        icon: "icons/smartchain.svg",
        chain: "Binance",
        price: getPrice("bsc"),
        marketCap: getMarketCap("bsc"),
        liquidity: getLiquidity("bsc"),
        reserves: getReserves("bsc"),
        holders: getHolders("bsc"),
        staked: getStaked("bsc"),
      });

      // Polygon
      rows.push({
        id: "poly",
        icon: "icons/polygon.svg",
        chain: "Polygon",
        price: getPrice("poly"),
        marketCap: getMarketCap("poly"),
        liquidity: getLiquidity("poly"),
        reserves: getReserves("poly"),
        holders: getHolders("poly"),
        staked: getStaked("poly"),
      });

      // Ethereum
      rows.push({
        id: "eth",
        icon: "icons/ethereum.svg",
        chain: "Ethereum",
        price: getPrice("eth"),
        marketCap: getMarketCap("eth"),
        liquidity: getLiquidity("eth"),
        reserves: getReserves("eth"),
        holders: getHolders("eth"),
        staked: getStaked("eth"),
      });

      // Fantom
      rows.push({
        id: "ftm",
        icon: "icons/fantom.svg",
        chain: "Fantom",
        price: getPrice("ftm"),
        marketCap: getMarketCap("ftm"),
        liquidity: getLiquidity("ftm"),
        reserves: getReserves("ftm"),
        holders: getHolders("ftm"),
        staked: getStaked("ftm"),
      });

      // Avalanche
      rows.push({
        id: "avax",
        icon: "icons/avalanche.svg",
        chain: "Avalanche",
        price: getPrice("avax"),
        marketCap: getMarketCap("avax"),
        liquidity: getLiquidity("avax"),
        reserves: getReserves("avax"),
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
      table.classList.add("table-dark");
      if (i === 0) {
        table.classList.add("table-bordered");
        table.querySelector("tbody").classList.add("text-warning");
      } else {
        table.classList.add("table-striped");
      }
    });

    updateTables();
  });
</script>

<div class="container text-center mt-4">
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

<div class="container text-start text-white mt-4">
  <details class:hidden={isLoading}>
    <summary>Details</summary>
    <pre id="json-formatted">{formattedData}</pre>
  </details>
</div>

<style>
  .hidden {
    visibility: hidden;
  }
  :global(.icon) {
    width: 16px;
    height: 16px;
  }

  #json-formatted {
    padding: 20px;
    border: 1px solid white;
  }
</style>
