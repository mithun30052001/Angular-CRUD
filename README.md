https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req-mapping.ts

export class ReqMappingEditDialogComponent {
  reqForm: FormGroup;
  jobs: Job[] = [];
  public managerComplete$: any;
  public spocComplete$: any;
  public managerControl = new FormControl<any>({ value: '', disabled: false }, [
    Validators.required,
    this.validateHrIdCtrl,
  ]);
  public spocControl = new FormControl<any>({ value: '', disabled: false }, [
    Validators.required,
    this.validateHrIdCtrl,
  ]);
  public jobControl = new FormControl<any>({ value: '' }, Validators.required);

  form: FormGroup = new FormGroup({
    reqForm: new FormGroup({
      jobId: this.jobControl,
      requisitionId: new FormControl(''),
      spoc: this.spocControl,
      closureDate: new FormControl(''),
      onboardingDate: new FormControl(''),
      manager: this.managerControl,
    }),
  });

  formFields = [
    {
      name: 'requisitionId',
      label: 'Requisition ID',
      placeholder: 'Enter requisition ID',
      type: 'text',
    },
    {
      name: 'closureDate',
      label: 'Closure Date',
      placeholder: 'Enter closure date',
      type: 'date',
    },
    {
      name: 'onboardingDate',
      label: 'Onboarding Date',
      placeholder: 'Enter onboarding date',
      type: 'date',
    },
  ];

  originalOpenNumbers: number | null = null; // Added to store the original openNumbers value
  openNumbersChanged = false; // Flag to track if openNumbers has changed

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

    const formattedOnboardingDate = '';
    if (rowData.onboardingDate) {
      const onboardingDateObject = new Date(rowData?.onboardingDate);
      const formattedOnboardingDate = onboardingDateObject?.toISOString().split('T')[0];
    }

    this.reqForm = this.fb.group({
      jobId: rowData.job.id || this.jobControl,
      requisitionId: [rowData?.requisitionId || '', Validators.required],
      spoc: this.spocControl,
      closureDate: [formattedClosureDate || '', Validators.required],
      onboardingDate: [formattedOnboardingDate || '', Validators.required],
      manager: this.managerControl,
      published: [true, Validators.required],
    });
    console.log('DIALOG DATA', rowData);

    if (rowData.openNumbers) {
      this.originalOpenNumbers = rowData.openNumbers; // Set the original value
      this.reqForm.addControl(
        'openNumbers',
        new FormControl(rowData.openNumbers, Validators.required)
      );
      this.reqForm.addControl('openNumbersProof', new FormControl(null));

      // Subscribe to changes on the openNumbers field
      this.reqForm.get('openNumbers')?.valueChanges.subscribe((newValue) => {
        this.openNumbersChanged = newValue !== this.originalOpenNumbers;
      });
    }
  }


req-mapping.html

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

<!-- The file uploader is only shown if openNumbers has changed -->
<div class="col-lg-4 col-sm-4" *ngIf="rowData.openNumbers && openNumbersChanged">
  <div class="form-group ags-form-group">
    <label for="openNumbersProof" class="form-label">
      Upload Proof<span class="required"></span>
    </label>
    <input
      type="file"
      class="form-control"
      formControlName="openNumbersProof"
      accept=".pdf,.png,.jpg,.jpeg"
    />
  </div>
</div>
