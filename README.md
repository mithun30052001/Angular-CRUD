https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


req-edit.ts

constructor(
  private dialogRef: MatDialogRef<ReqMappingEditDialogComponent>,
  @Inject(MAT_DIALOG_DATA) public rowData: any,
  private reqService: ReqService,
  private fb: FormBuilder,
  private applicationService: ApplicationService,
  private authService: AuthService
) {
  const closureDateObject = new Date(rowData?.closureDate);
  const formattedClosureDate = closureDateObject.toISOString().split('T')[0];
  
  const formattedOnboardingDate = rowData.onboardingDate 
    ? new Date(rowData?.onboardingDate).toISOString().split('T')[0] 
    : '';

  this.reqForm = this.fb.group({
    jobId: rowData.job.id || this.jobControl,
    requisitionId: [rowData?.requisitionId || '', Validators.required],
    spoc: this.spocControl,
    closureDate: [formattedClosureDate || '', Validators.required],
    onboardingDate: [formattedOnboardingDate || '', Validators.required],
    manager: this.managerControl,
    published: [true, Validators.required],
  });

  // Conditionally add openNumbers field
  if (rowData.openNumbers) {
    this.reqForm.addControl('openNumbers', new FormControl(rowData.openNumbers, Validators.required));
    this.reqForm.addControl('openNumbersProof', new FormControl(null));
  }

  console.log('DIALOG DATA', rowData);
}

ngOnInit(): void {
  console.log("MY DIALOG DATA INIT", this.rowData);
  this.getJobs();

  // Enable file upload only when openNumbers value is changed
  if (this.rowData.openNumbers) {
    this.reqForm.get('openNumbers')?.valueChanges.subscribe((newValue) => {
      const proofControl = this.reqForm.get('openNumbersProof');
      if (newValue !== this.rowData.openNumbers) {
        proofControl?.setValidators(Validators.required);
      } else {
        proofControl?.clearValidators();
      }
      proofControl?.updateValueAndValidity();
    });
  }

  this.managerComplete$ = this.managerControl.valueChanges.pipe(
    startWith(''),
    debounceTime(300),
    switchMap((value) => {
      if (value !== '') {
        if (value?.fullName) {
          return of(null);
        }
        return this.lookup(value);
      } else {
        return this.lookup('');
      }
    })
  );

  this.spocComplete$ = this.spocControl.valueChanges.pipe(
    startWith(''),
    debounceTime(300),
    switchMap((value) => {
      if (value !== '') {
        if (value?.fullName) {
          return of(null);
        }
        return this.lookup(value);
      } else {
        return this.lookup('');
      }
    })
  );
}

onSubmit() {
  if (this.reqForm.valid) {
    const formValues = this.reqForm.value;
    console.log('Open Numbers:', formValues.openNumbers);
    console.log('Proof File:', formValues.openNumbersProof);
    formValues.manager = formValues.manager.id;
    formValues.spoc = formValues.spoc.id;

    this.reqService.updateJobRequisition(formValues).subscribe(
      (response) => {
        console.log('Job requisition updated successfully:', response);
        this.dialogRef.close(true);
      },
      (error) => {
        console.error('Error updating job requisition:', error);
      }
    );
  }
}


req-edit.html

<form [formGroup]="reqForm" (ngSubmit)="onSubmit()">
  <div class="req-edit-body">
    <div class="row">
      <!-- Other form fields -->
      
      <!-- Open Numbers Field -->
      <div class="col-lg-4 col-sm-4" *ngIf="rowData.openNumbers">
        <div class="form-group ags-form-group">
          <label for="openNumbers" class="form-label">
            Open Numbers<span class="required"></span>
          </label>
          <mat-form-field>
            <input
              matInput
              formControlName="openNumbers"
              type="number"
              placeholder="Enter open numbers"
              required
            />
          </mat-form-field>
        </div>
      </div>

      <!-- File Upload Proof -->
      <div class="col-lg-4 col-sm-4" *ngIf="rowData.openNumbers">
        <div class="form-group ags-form-group">
          <label for="openNumbersProof" class="form-label">
            Upload Proof<span class="required" *ngIf="reqForm.get('openNumbersProof')?.hasValidator(Validators.required)"></span>
          </label>
          <input
            type="file"
            class="form-control"
            formControlName="openNumbersProof"
            accept=".pdf,.png,.jpg,.jpeg"
          />
        </div>
      </div>
    </div>
  </div>

  <div class="req-edit-footer">
    <div>
      <button
        title="Cancel"
        mat-dialog-close
        class="ags-outline-btn ags-hmd44 btn-font16 ags-padding1624"
      >
        Cancel
      </button>
    </div>
    <div>
      <button
        title="Submit"
        type="submit"
        class="ags-primary-btn ags-hmd44 btn-font16 ags-padding1624"
        [disabled]="reqForm.invalid"
      >
        Publish
      </button>
    </div>
  </div>
</form>
