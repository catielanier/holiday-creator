<script>
  import { DateTime } from "luxon";
  let holidays = [{ name: "", date: "" }];
  const generateHolidayCSV = () => {
    if (
      holidays[holidays.length - 1].name === "" &&
      holidays[holidays.length - 1].date === ""
    )
      holidays.pop();
    holidays.sort((x, y) => {
      return x.date.localeCompare(y.date);
    });
    const csv = [["Date", "Reason"]];
    const filename = `Holiday Calendar - ${DateTime.now().toFormat(
      "MMM d, yyyy"
    )}`;
    holidays.forEach((holiday) => {
      const row = [holiday.date, holiday.name];
      csv.push(row);
    });
    let csvContent =
      "data:text/csv;charset=utf-8," + csv.map((e) => e.join(",")).join("\n");
    const encodedUri = encodeURI(csvContent);
    const link = document.createElement("a");
    link.setAttribute("href", encodedUri);
    link.setAttribute("download", `${filename}.csv`);
    document.body.appendChild(link);
    link.click();
  };
  const removeHoliday = (index) => {
    if (holidays.length > 1) {
      const oldHolidays = holidays;
      oldHolidays.splice(index, 1);
      holidays = oldHolidays;
    }
  };
  const parseHolidaysFromExistingCSV = async (e) => {
    const csvHolidays = [];
    const { files } = e.target || e.dataTransfer;
    const file = await files[0];
    const reader = new FileReader();
    let csvData;
    const promise = new Promise((resolve, reject) => {
      reader.onload = async (e) => {
        resolve((csvData = reader.result));
      };
      reader.readAsText(file);
    });
    promise
      .then((res) => {
        const lines = csvData.split(/\r\n|\n/);
        lines.pop();
        const arr = [];
        lines.forEach((line) => {
          const arr2 = line.split(",");
          arr.push(arr2);
        });
        arr.forEach((line, index) => {
          if (index > 0) {
            const newHoliday = {
              date: line[0],
              name: line[1],
            };
            csvHolidays.push(newHoliday);
          }
        });
        csvHolidays.push({ name: "", date: "" });
        csvHolidays.sort((x, y) => {
          return x.date.localeCompare(y.date);
        });
        holidays = csvHolidays;
      })
      .catch((err) => {
        console.error(err);
      });
  };
  const addRowToHolidays = () => {
    const oldHolidays = holidays;
    oldHolidays.push({ name: "", date: "" });
    holidays = oldHolidays;
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
  <div>
    <button on:click={addRowToHolidays}>+</button>
  </div>
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
