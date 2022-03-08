<script>
  import { DateTime } from "luxon";
  let holidays = [{ name: "", date: "" }];
  const generateHolidayCSV = () => {};
  const parseHolidaysFromExistingCSV = () => {
    const csvHolidays = [];
    // do shit
    csvHolidays.push({ name: "", date: "" });
    holidays = csvHolidays;
  };
  const convertDateToTimestamp = (date, index) => {
    const timeStampRegex = new RegExp(/([0-9]{4}-[0-9]{2}-[0-9]{2})/);
    let timestamp;
    const isTimestamp = timeStampRegex.test(date);
    if (!isTimestamp) {
      const monthDayYearSlashRegex = new RegExp(
        /([0-9]{2}\/[0-9]{2}\/[0-9]{4})/
      );
      const monthDayYearSpelledRegex = new RegExp(
        /([A-Z][a-z]* [0-9]{2}, [0-9]{4})/
      );
      const isSlashDate = monthDayYearSlashRegex.test(date);
      const isWordDate = monthDayYearSpelledRegex.test(date);
      if (isSlashDate) {
        timestamp = DateTime.fromFormat(date, "MM/dd/yyyy").toISODate();
      }
      if (isWordDate) {
        const medWordDateRegex = new RegExp(
          /([A-Z][a-z]{2} [0-9]{2}, [0-9]{4})/
        );
        const isMedWordDate = medWordDateRegex.test(date);
        if (isMedWordDate) {
          timestamp = DateTime.fromFormat(date, "MMM d, yyyy").toISODate();
        } else {
          timestamp = DateTime.fromFormat(date, "MMMM d, yyyy").toISODate();
        }
      }
      holidays[index].date = timestamp;
    }
  };
</script>

<main>
  <h1>Holiday CSV Generator</h1>
  <p>Update existing CSV:</p>
  <input
    type="file"
    on:change={parseHolidaysFromExistingCSV}
    multiple="false"
    accept=".csv"
  />
  {#each holidays as holiday, index}
    <div>
      <div>
        <input
          type="text"
          bind:value={holiday.date}
          on:blur={() => convertDateToTimestamp(holiday.date, index)}
          placeholder="Date"
        />
      </div>
      <div>
        <input type="text" bind:value={holiday.name} placeholder="Name" />
      </div>
    </div>
  {/each}
  <button on:click={generateHolidayCSV}>Generate CSV</button>
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
