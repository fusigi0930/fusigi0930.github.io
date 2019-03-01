# Calendar Event Update
## description
in this case, I used Google extend service api to get which day event is updated
while onEventUpdated is triggered.

because the trigger event is not contain calendar event information, (e.g.)
{calendarId=xxx@gmail.com, authMode=FULL, triggerUid=615558}

we need find it by ourself, the code will show all of events title in the update date

## pre-condition
1. the trigger event are already registered with function - triggerCalUpdate
2. all of permissions are allowed
3. the "Google Calendar API" is enabled in your console.cloud.google.com

## code
```javascript
function triggerCalUpdate(trigger_event) {
  // get the update date from calendar
  var calendar = CalendarApp.getDefaultCalendar();
  var update_day = Calendar.Events.list(trigger_event.calendarId).updated;
  var day = new Date(update_day);
  console.info('update date: %s', day);

  var events = calendar.getEventsForDay(day);
  for (var i=0; i<events.length; i++) {
    console.info('event title: %s', events[i].getTitle());
  }
}
```

## use case
1. Goto your calnedar.google.com, then add a event
2. Edit some event from your calendar
3. Remove event from your calendar

you can see the log to verify the result.
