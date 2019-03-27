---
layout: post
title: Google Script - calendar
description: calendar for google script
tags: google-script javascript
---

# Calendar Event Update
## description
in this case, I used Google extend service api to get which day event is updated
while onEventUpdated is triggered.

because the trigger event is not contain calendar event information, (e.g.)
{calendarId=xxx@gmail.com, authMode=FULL, triggerUid=615558}

we need find it by ourself, the code will use the calendar cloud api to implemented.
it will get the last updated event's title.

## pre-condition
1. the trigger event are already registered with function - triggerCalUpdate
2. all of permissions are allowed
3. the "Google Calendar API" is enabled in your console.cloud.google.com

## code
```javascript
function triggerCalUpdate(trigger_event) {
  var day = new Date();
  var one_day = 24 * 60 * 60 * 1000;
  var start_day = new Date(day.getTime() - 360 * one_day);
  var end_day = new Date(day.getTime() + 360 * one_day);

  // set the events range are between one year ago to one year later
  var str_start = Utilities.formatDate(start_day, "GMT+8", "yyyy-MM-dd'T'HH:mm:ss'Z'");
  var str_end = Utilities.formatDate(end_day, "GMT+8", "yyyy-MM-dd'T'HH:mm:ss'Z'");

  var resource = {
    "orderBy" : "updated",
    "timeMin" : str_start,
    "timeMax" : str_end,
  };

  var events = Calendar.Events.list(triggerEvent.calendarId, resource).items;
  var index = events.length - 1;

  var calendar = CalendarApp.getDefaultCalendar();
  var event = calendar.getEventById(events[index].iCalUID);

  console.info('event title: %s', event.getTitle());
}
```

## use case
1. Goto your calnedar.google.com, then add a event
2. Edit some event from your calendar

you can see the log to verify the result.
