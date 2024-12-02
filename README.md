https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

this.filteredTimeZoneList.subscribe((timezones) => {
    if (this.eventData?.meta?.timezone) {
      const matchedTimezone = timezones.find((timezone: any) => timezone.name === this.eventData.meta.timezone);
      if (matchedTimezone) {
        console.log('Matching Timezone found:', matchedTimezone);
        this.panelForm.patchValue({
          timezone: matchedTimezone
        });
      }
    }
  });
