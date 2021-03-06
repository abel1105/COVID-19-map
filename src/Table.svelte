<script>
  import { format } from 'd3-format';

  export let data;
  export let text;

  let fullMode = false;

  let updatedTime;

  const toggleFullMode = () => { fullMode = !fullMode };

  const renderTime = (timestamp) => {
    return new Date(timestamp).toLocaleString()
  };

  const formatter = format(",");

  const groupDataByCountry = data.reduce((acc, row) => {
    const country = row['Country/Region'];
    const time = +new Date(`${row['Last Update'].split(' ').join('T')}Z`);
    // ignore 0 confirmed and 0 recovered country
    // Canada and US recovered data are separated
    if (!parseInt(row.Confirmed) && !parseInt(row.Recovered)) {
      return acc
    }
    if (acc[country]) {
      acc[country] = {
        ...acc[country],
        items: [
          ...acc[country].items,
          row
        ],
        Confirmed: acc[country].Confirmed + parseInt(row.Confirmed),
        Treatment: acc[country].Treatment + parseInt(row.Treatment),
        Recovered: acc[country].Recovered + parseInt(row.Recovered),
        Deaths: acc[country].Deaths + parseInt(row.Deaths)
      }
    }else {
      acc[country] = {
        items: [ row ],
        Confirmed: parseInt(row.Confirmed),
        Treatment: parseInt(row.Treatment),
        Recovered: parseInt(row.Recovered),
        Deaths: parseInt(row.Deaths),
        country: country
      }
    }
    if (!updatedTime || (updatedTime && updatedTime < time)) {
      updatedTime = time;
    }
    return acc
  }, {});

  const sortCountry = Object.keys(groupDataByCountry)
    .map(country => groupDataByCountry[country])
    .sort((a, b) => b.Confirmed - a.Confirmed)
    .map(country => {
      return {
        ...country,
        items: country.items.sort((a, b) => b.Confirmed - a.Confirmed)
      }
    })
</script>

<div>
  <h2 class="title">{text.tableTitle}</h2>
  <div class="time">{text.updatedTime}{renderTime(updatedTime)}</div>
  <div class="button" on:click="{toggleFullMode}">
    <i class="material-icons toggle">{fullMode ? "check_box" : "check_box_outline_blank"}</i><span>{text.toggleBtn}</span>
  </div>
  <div class="table-wrapper">
    <div class="table" class:full={fullMode}>
      <div class="cell header">#</div>
      <div class="cell header">{text.lang.country}</div>
      {#if fullMode}<div class="cell header">{text.lang.state}</div>{/if}
      <div class="cell header number confirm">{text.lang.confirmed}</div>
      <div class="cell header number treat">{text.lang.treatment}</div>
      <div class="cell header number recover">{text.lang.recovered}</div>
      <div class="cell header number death">{text.lang.deaths}</div>
      {#each sortCountry as row, index}
        <div class="cell" class:taiwan={row.country === '台灣'}>{index + 1}</div>
        <div class="cell" class:taiwan={row.country === '台灣'}>{row.country}</div>
        {#if fullMode}<div class="cell" class:taiwan={row.country === '台灣'}></div>{/if}
        <div class="cell number confirm">{formatter(row.Confirmed)}</div>
        <div class="cell number treat">{row.Treatment < 0 ? 0 : formatter(row.Treatment)}</div>
        <div class="cell number recover">{formatter(row.Recovered)}</div>
        <div class="cell number death">{formatter(row.Deaths)}</div>
        {#if fullMode && row.items.length > 1}
          {#each row.items as item}
            <div class="cell"></div>
            <div class="cell"></div>
            <div class="cell">{item['Province/State'] ? item['Province/State'] : row.country}</div>
            <div class="cell number confirm">{formatter(item.Confirmed)}</div>
            <div class="cell number treat">{item.Treatment < 0 ? 0 : formatter(item.Treatment)}</div>
            <div class="cell number recover">{formatter(item.Recovered)}</div>
            <div class="cell number death">{formatter(item.Deaths)}</div>
          {/each}
        {/if}
      {/each}
    </div>
  </div>
</div>

<style>
.title {
  color: rgba(255, 255, 255, 1);
}
.time {
  color: rgba(255, 255, 255, 0.6);
  margin-bottom: 16px;
}
.button {
  display: flex;
  justify-content: center;
  align-items: center;
  color: rgba(255, 255, 255, 0.6);
  font-size: 18px;
  cursor: pointer;
  margin-bottom: 16px;
}
.toggle {
  font-size: 24px;
  color: rgba(255, 255, 255, 1);
  margin-right: 8px;
}
.table {
  color: #fff;
  display: grid;
  grid-template-columns: 60px repeat(5, minmax(auto, auto));
  grid-gap: 1px;
  min-width: 500px;
  max-width: 999px;
  margin: auto;
  background: rgba(255, 255, 255, 0.12);
}
.table.full {
  grid-template-columns: 60px repeat(6, minmax(auto, auto));
}
.cell {
  padding: 4px 16px;
  height: 52px;
  background: #121212;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  text-align: left;
}
.header {
  color: rgba(255, 255, 255, 0.6);
  height: 56px !important;
  position: sticky;
  top: 0;
}
.taiwan {
  background: rgba(18, 18, 18, 0.5);
}
.number {
  justify-content: flex-end;
}
.confirm {
  background: rgb(58, 45, 46);
}
.treat {
  background: rgb(63, 57, 51);
}
.recover {
  background: rgb(59, 66, 50);
}
.death {
  background: rgb(44, 44, 44);
}
@media (max-width: 500px) {
  .button {
    display: none;
  }
  .table-wrapper {
    overflow: auto;
  }
}

</style>

