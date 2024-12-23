https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

 setInitialValues() {
    let interview = this.summary.interviewProcess;
    interview = interview[interview.length - 1];
    const candidate = interview?.candidates[0];

    if (interview != null) {
      this.form.patchValue({
        feedBack: candidate?.feedback ?? '',
      });

      const initialStatus = this.summary?.applicationStatus;
      this.form.patchValue({
        status: initialStatus ?? '',
      });
    }

    if (
      this.form.value.status === 'PANEL_SELECTED' ||
      this.form.value.status === 'PANEL_REJECTED'
    ) {
      this.formSubmitted = true;
    }

    if (this.formSubmitted) {
      this.disableForm();
    }
  }

  disableForm() {
    this.form.get('feedBack')?.disable();
    this.form.get('status')?.disable();
  }
