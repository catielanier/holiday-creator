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
    const csv = [["date", "reason"]];
    const filename = `Holiday Calendar - ${DateTime.now().toFormat(
      "MMM d, yyyy"
    )}`;
    holidays.forEach((holiday) => {
      const row = [
        holiday.date,
        holiday.name.replace(/[`~!@#$%^&*()_|+\-=?;:'",.<>\{\}\[\]\\\/]/gi, ""),
      ];
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
    const oldHolidays = holidays;
    oldHolidays.splice(index, 1);
    if (!oldHolidays.length) {
      oldHolidays.push({ name: "", date: "" });
    }
    holidays = oldHolidays;
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
        csvHolidays.sort((x, y) => {
          return x.date.localeCompare(y.date);
        });
        csvHolidays.push({ name: "", date: "" });
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
        /([0-9]{1,2}\/[0-9]{1,2}\/[0-9]{2,4})/
      );
      const monthDayYearSpelledRegex = new RegExp(
        /([A-Za-z]*.{0,1} [0-9]{1,2},{0,1} [0-9]{2,4})/
      );
      const dayMonthYearSpelledRegex = new RegExp(
        /[0-9]{1,2} ([A-Za-z]*.{0,1},{0,1} [0-9]{2,4})/
      );
      const isSlashDate = monthDayYearSlashRegex.test(date);
      if (isSlashDate) {
        const twoDigitYearRegex = new RegExp(
          /([0-9]{1,2}\/[0-9]{1,2}\/[0-9]{2})/
        );
        const isTwoDigitYear = twoDigitYearRegex.test(date);
        if (isTwoDigitYear) {
          timestamp = DateTime.fromFormat(date, "M/d/yy").toISODate();
        } else {
          timestamp = DateTime.fromFormat(date, "M/d/yyyy").toISODate();
        }
      }
      const isWordDate =
        monthDayYearSpelledRegex.test(date) ||
        dayMonthYearSpelledRegex.test(date);
      if (isWordDate) {
        const wordDate = date.replace(",", "").replace(".", "");
        const isDayFirst = dayMonthYearSpelledRegex.test(date);
        if (isDayFirst) {
          const medWordDateRegex = new RegExp(
            /([0-9]{1,2} [A-Za-z]{3} [0-9]{2,4})/
          );
          const isMedWordDate = medWordDateRegex.test(wordDate);
          const twoDigitYearRegex = new RegExp(
            /([0-9]{1,2} [A-Za-z]* [0-9]{2})/
          );
          const isTwoDigitYear = twoDigitYearRegex.test(wordDate);
          switch (isTwoDigitYear) {
            case true:
              if (isMedWordDate) {
                timestamp = DateTime.fromFormat(
                  wordDate,
                  "d MMM yy"
                ).toISODate();
              } else {
                timestamp = DateTime.fromFormat(
                  wordDate,
                  "d MMMM yy"
                ).toISODate();
              }
              break;
            default:
              if (isMedWordDate) {
                timestamp = DateTime.fromFormat(
                  wordDate,
                  "d MMM yyyy"
                ).toISODate();
              } else {
                timestamp = DateTime.fromFormat(
                  wordDate,
                  "d MMMM yyyy"
                ).toISODate();
              }
          }
        } else {
          const medWordDateRegex = new RegExp(
            /([A-Za-z]{3} [0-9]{1,2} [0-9]{2,4})/
          );
          const isMedWordDate = medWordDateRegex.test(wordDate);
          const twoDigitYearRegex = new RegExp(
            /([A-Za-z]* [0-9]{1,2} [0-9]{2})/
          );
          const isTwoDigitYear = twoDigitYearRegex.test(wordDate);
          switch (isTwoDigitYear) {
            case true:
              if (isMedWordDate) {
                timestamp = DateTime.fromFormat(
                  wordDate,
                  "MMM d yy"
                ).toISODate();
              } else {
                timestamp = DateTime.fromFormat(
                  wordDate,
                  "MMMM d yy"
                ).toISODate();
              }
              break;
            default:
              if (isMedWordDate) {
                timestamp = DateTime.fromFormat(
                  wordDate,
                  "MMM d yyyy"
                ).toISODate();
              } else {
                timestamp = DateTime.fromFormat(
                  wordDate,
                  "MMMM d yyyy"
                ).toISODate();
              }
          }
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
  <div class="holiday-table table-header">
    <div>Date</div>
    <div>Holiday Name</div>
  </div>
  {#each holidays as holiday, index}
    <div class="holiday-table">
      <div>
        <input
          type="text"
          bind:value={holiday.date}
          on:blur={() => convertDateToTimestamp(holiday.date, index)}
          placeholder="Date"
        />
      </div>
      <div class="name-and-controls">
        <div>
          <input
            type="text"
            bind:value={holiday.name}
            placeholder="Name"
            class="holiday-name-input"
          />
        </div>
        <div>
          <button class="control minus" on:click={() => removeHoliday(index)}
            >-</button
          >
        </div>
        <div>
          {#if index === holidays.length - 1}
            <button class="control plus" on:click={addRowToHolidays}>+</button>
          {/if}
        </div>
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
    background: #473335;
    color: #f7f7ff;
  }

  h1 {
    color: #fcaa67;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  .holiday-table.table-header {
    margin-top: 15px;
  }

  .holiday-name-input {
    width: 70%;
  }
  .name-and-controls {
    display: grid;
    grid-template-columns: 11fr 1fr 1fr;
    grid-gap: 10px;
  }

  .holiday-table {
    max-width: 540px;
    width: 100%;
    margin: 0 auto;
    margin-bottom: 15px;
    display: grid;
    grid-template-columns: 30% 70%;
    grid-gap: 15px;
  }
  .holiday-table div {
    text-align: left;
  }

  .holiday-table input {
    width: 100%;
  }

  .table-header {
    font-weight: bold;
  }

  button {
    border: 0;
    border-radius: 5px;
    background: #fcaa67;
    color: #f7f7ff;
  }

  .control {
    width: 25px;
    height: 25px;
    border-radius: 50%;
    font-weight: bold;
    padding: 2.5px;
  }

  .plus {
    background: #3f826d;
    color: #f7f7ff;
  }

  .minus {
    background: #e3170a;
    color: #f7f7ff;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
