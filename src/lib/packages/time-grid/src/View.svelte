<script>
    import { getContext } from "svelte";
    import { setContent, toISOString } from "@event-calendar/core";
    import Section from "./Section.svelte";
    import Body from "./Body.svelte";
    import Day from "./Day.svelte";
    import Week from "./all-day/Week.svelte";

    let { _events, _viewDates, _intlDayHeader, _intlDayHeaderAL, allDaySlot, theme } = getContext("state");

    let holidays = getContext("holidays");

    function checkHolidays(date) {
       const year = date.getFullYear().toString().padStart(4, '0');
       const month = (date.getMonth() + 1).toString().padStart(2, '0');
       const day = date.getDate().toString().padStart(2, '0');
       const yyyymmdd = year + "/" + month + "/" + day
       for (const e of holidays) {
            if (yyyymmdd == e.date) {
               date.holiday = true;
               date.holiday_name = e.name;
               return date;
            }
       }
       date.holiday = false;
       return date;
    }
/*
    let dates = [];
    for (let i = 0; i < $_viewDates.length ; ++i) {
            let day = $_viewDates[i];
            let day_ = checkHolidays(day);
            dates.push(day_);
    }
*/
    let dates = $derived.by(() => {
          let dates_ = [];
          for (let i = 0; i < $_viewDates.length ; ++i) {
                  let day = $_viewDates[i];
                  let day_ = checkHolidays(day);
                  dates_.push(day_);
          }
          return dates_;
    });
    //let isHoliday = true;
    //let holidayName = "TEST";
    //console.log(dates);
</script>

<div class={$theme.header}>
    <Section>
    <!--
        {#each $_viewDates as date}
	-->
        {#each dates as date}
            <div class="{$theme.day} {$theme.weekdays?.[date.getUTCDay()]} {date.holiday ? 'ec-day-holiday' : ' '}" role="columnheader">
                <time
                    datetime={toISOString(date, 10)}
                    aria-label={$_intlDayHeaderAL.format(date)}
                    use:setContent={$_intlDayHeader.format(date)}
                ></time>
                {#if date.holiday}
                     <span>{date.holiday_name}</span>
                {/if}
            </div>
        {/each}
    </Section>
    <div class={$theme.hiddenScroll}></div>
</div>
{#if $allDaySlot}
    <div class={$theme.allDay}>
        <div class={$theme.content}>
            <Section>
                <Week dates={$_viewDates} />
            </Section>
            <div class={$theme.hiddenScroll}></div>
        </div>
    </div>
{/if}
<Body>
    {#each $_viewDates as date}
        <Day {date} />
    {/each}
</Body>
