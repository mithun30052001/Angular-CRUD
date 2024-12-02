https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

initForm()

initForm(): void {
  const closureDateObject = this.rowData?.closureDate ? new Date(this.rowData?.closureDate) : new Date();
  const formattedClosureDate = closureDateObject.toISOString().split('T')[0];
  
  const onboardingDateObject = this.rowData?.onboardingDate ? new Date(this.rowData?.onboardingDate) : new Date();
  const formattedOnboardingDate = onboardingDateObject.toISOString().split('T')[0];
  
  console.log("MY FORM DATA INSIDE INITFORM", this.rowData);
  
  this.reqForm = this.fb.group({
    jobId: this.rowData?.job?.id ?? this.jobControl,
    requisitionId: [this.rowData?.requisitionId ?? '', Validators.required],
    spoc: this.spocControl,
    closureDate: [formattedClosureDate, Validators.required],  // Use current date if not present
    onboardingDate: [formattedOnboardingDate, Validators.required],  // Use current date if not present
    manager: this.managerControl,
    published: [true, Validators.required],
  });

  if (this.rowData?.hrSpoc != null) {
    this.reqForm.patchValue({ spoc: this.rowData?.hrSpoc });
  }

  if (this.rowData?.hrManager != null) {
    this.reqForm.patchValue({ manager: this.rowData?.hrManager });
  }

  if (this.isHiringUpdate === true) {
    this.originalOpenNumbers = this.rowData.openNumbers;
    this.reqForm.addControl(
      'openNumbers',
      new FormControl(this.rowData.openNumbers, Validators.required)
    );
    this.reqForm.addControl('openNumbersProof', new FormControl(null));

    this.reqForm.get('openNumbers')?.valueChanges.subscribe((newValue) => {
      this.openNumbersChanged = newValue !== this.originalOpenNumbers;
    });
  } else {
    this.formFields.unshift({
      name: 'requisitionId',
      label: 'Requisition ID',
      placeholder: 'Enter requisition ID',
      type: 'text',
    });
  }
}
