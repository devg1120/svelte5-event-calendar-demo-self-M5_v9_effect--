<script>
    import { untrack } from "svelte";

    import { getContext } from "svelte";
    import Week from "./Week.svelte";

    let { _bodyEl, _viewDates, _hiddenEvents, dayMaxEvents, hiddenDays, theme } = getContext("state");

    let holidays = getContext("holidays");
    //console.log("holidays", holidays)
    //console.log("_viewDates", _viewDates)


    let weeks = $state();
    let days = $state();

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
       return date;
    }

    $effect.pre(() => {
        weeks = [];
        days = 7 - $hiddenDays.length;
        $_hiddenEvents = {};
        $dayMaxEvents;
        for (let i = 0; i < $_viewDates.length / days; ++i) {
            let dates = [];
            for (let j = 0; j < days; ++j) {
                //dates.push($_viewDates[i * days + j]);
                let day = $_viewDates[i * days + j];
		//console.log(typeof(day));
		//console.log(day.getDate());
		/*
		if (j ==3 ) {
		  day.holiday = true;
		  day.holiday_name = "テストの日";
		}
		*/
		day = checkHolidays(day);
                dates.push(day);

            }
            untrack(() => {
                weeks.push(dates);
            });
        }
    });
</script>

<div bind:this={$_bodyEl} class="{$theme.body}{$dayMaxEvents === true ? ' ' + $theme.uniform : ''}">
    <div class={$theme.content}>
        {#each weeks as dates}
            <Week {dates} />
        {/each}
    </div>
</div>
