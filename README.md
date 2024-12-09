https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

ngOnInit(): void {
  this.initiateForm();
  this.setInitialValues();
}

initiateForm() {
  this.form = new FormGroup({
    feedBack: new FormControl('', Validators.required),
    status: new FormControl('', Validators.required),
  });
}

setInitialValues() {
  const interview = this.summary.interviewProcess;
  const candidate = interview?.candidates[0];  // Make sure this exists
  
  if (candidate) {
    // Set the initial value for 'feedBack'
    this.form.patchValue({
      feedBack: candidate.feedback || '', // Ensure feedback is set
    });

    // Set the initial value for 'status' using the getStatus method
    const initialStatus = this.getStatus(candidate.interviewStatus);
    this.form.patchValue({
      status: initialStatus || '', // Ensure status is set
    });
  }
}
