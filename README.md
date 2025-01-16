https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

listenForValueChanges() {
    // primary and secondary panel members autocomplete
    this.primaryPanelAc$ = this.primaryPanelAcControl.valueChanges.pipe(
      startWith(''),
      // delay emits
      debounceTime(300),
      // use switch map so as to cancel previous subscribed events, before creating new once
      switchMap((value) => {
        if (value !== '') {
          if (value?.employeeName) {
            return of(null);
          }

          return this.panelLookup(value);
        } else {
          // if no value is present, return null
          return this.panelLookup('');
        }
      })
    );

    this.secondaryPanelAc$ = this.secondaryPanelAcControl.valueChanges.pipe(
      startWith(''),
      debounceTime(300),
      switchMap((value) => {
        if (value !== '') {
          if (value?.employeeName) {
            return of(null);
          }

          return this.panelLookup(value);
        } else {
          // if no value is present, return null
          return this.panelLookup('');
        }
      })
    );

    // job and candidate list autocomplete
    if (this.data?.job?.jobId == null || this.data?.job?.jobId == undefined) {
      this.jobAc$ = this.jobAcControl.valueChanges.pipe(
        startWith(''),
        // delay emits
        debounceTime(300),
        // use switch map so as to cancel previous subscribed events, before creating new once
        switchMap((value) => {
          if (value !== '') {
            if (value?.jobTitle) {
              return of(null);
            }

            return this.jobLookup(value);
          } else {
            // if no value is present, return null
            return this.jobLookup('');
          }
        })
      );
    }

    this.applicationAc$ = this.applicationAcControl.valueChanges.pipe(
      startWith(''),
      debounceTime(300),
      switchMap((value) => {
        if (value !== '') {
          if (value?.name) {
            return of(null);
          }

          return this.applicationLookup(value);
        } else {
          // if no value is present, return null
          return this.applicationLookup('');
        }
      })
    );
  }
