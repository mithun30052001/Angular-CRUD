https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

panel-schedule.component.ts

export class PanelScheduleDialogComponent
  extends ResponsiveViewComponent
  implements OnInit
{
  applicationList: {
    applicationId: string;
    email?: string;
    fullName?: string;
    mobile?: string;
  }[] = [];

  min = new Date();
  //timezone = Intl.DateTimeFormat().resolvedOptions().timeZone;

  timingList: any[] = [];
  timeZoneList: any[] = [];
  panelMembers: PanelMember[] = [];
  secondaryPanelMembers: PanelMember[] = [];
  panelForm: FormGroup;
  hrId = '';
  isAvailableTimeChecked: boolean;
  filteredTimeZoneList: Observable<any[]>;
  lastFilter: string = '';
  jobs: {
    jobTitle: string;
    location: string;
    id: string;
    display: string;
    experienceLevel: string;
  }[] = [];
  jobId: string;
  options: any = [];
  public jobDetails: any[] = [];

  get f() {
    return this.panelForm.controls;
  }

  public primaryPanelAc$: any;
  public primaryPanelAcControl = new FormControl<any>(
    { value: '', disabled: false },
    [Validators.required, this.validateEmployeeIdCtrl]
  );
  public secondaryPanelAcControl = new FormControl<any>({
    value: '',
    disabled: false,
  });
  public secondaryPanelAc$: any;

  public jobAc$: any;
  public jobAcControl = new FormControl<any>({ value: '', disabled: false }, [
    Validators.required,
    this.validateJobIdCtrl,
  ]);
  public reqAcControl: FormControl<any> = new FormControl<any>({ value: '' }, [
    Validators.required,
  ]);
  public reqAc$: any;
  public applicationAcControl = new FormControl<any>({
    value: '',
    disabled: true,
  });
  public applicationAc$: any;

  selectedPanel: PanelMember[] = [];
  selectedCandidates: any[] = [];

  @ViewChild('secAutoCompleteInput')
  secAutoCompleteInput: ElementRef<HTMLInputElement>;

  @ViewChild('candidateAutoCompleteInput')
  candidateAutoCompleteInput: ElementRef<HTMLInputElement>;

  isEditMode: boolean = false;
  dialogHeading: string = 'Panel Schedule';
  scheduleButtonText: string = 'Schedule';
  eventData: any;

  constructor(
    media: MediaMatcher,
    @Inject(MAT_DIALOG_DATA) private data: any,
    private formBuilder: FormBuilder,
    private applicationService: ApplicationService,
    private reqService: ReqService,
    private jobService: JobService,
    private dialog: MatDialog,
    public dialogRef: MatDialogRef<PanelScheduleDialogComponent>
  ) {
    super(media);
    this.isAvailableTimeChecked = false;
    if (data.job?.jobId) this.jobId = data.job?.jobId;

    if (this.jobId) {
      var reqIdList: any = [];
      this.reqService
        .getPublishedRequisitions({
          jobId: this.jobId,
        })
        .subscribe((res) => {
          res.forEach((x: { requisitionId: any }) => {
            reqIdList.push(x);
          });
          this.reqAc$ = reqIdList;
        });
    }
    this.hrId = localStorage.getItem('id') as string;
    var timezoneslist = getTimeZones({ includeUtc: true });
    var countryNames = ['Mexico', 'United States', 'Philippines', 'India'];
    var items = timezoneslist.filter((x) =>
      countryNames.includes(x.countryName)
    );
    items.forEach((tz) => {
      timezoneslist.splice(timezoneslist.indexOf(tz), 1);
      if (tz.countryName !== 'India') timezoneslist.unshift(tz);
    });

    timezoneslist.unshift(items?.filter((x) => x.countryName == 'India')[0]);

    this.timeZoneList = timezoneslist;
    this.eventData = data;
    console.log('EVENTDATA INSIDE CONSTRUCTOR', this.eventData);
  }
  filterMethod(event: any) {
    this.applicationService.getPanelMembersV2(event.query).subscribe((res) => {
      this.options = res;
    });
  }
}



form-fields in panel-schedule.html

 <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="assignor" class="form-label"
              >Candidates<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select
                placeholder="Select Candidates"
                formControlName="applicationIds"
                multiple
              >
                <mat-option
                  [title]="application.email"
                  *ngFor="let application of applicationList"
                  [value]="application.applicationId"
                  >{{ application.fullName }}{{application.mobile}}</mat-option
                >
              </mat-select> -->
              <mat-chip-grid
                #candidateChipGrid
                aria-label="Candidate selection"
              >
                <mat-chip-row
                  *ngFor="let item of selectedCandidates"
                  (removed)="removec(item.id)"
                >
                  {{ item.name }} ({{ item.mobile }})
                  <button matChipRemove [attr.aria-label]="'remove ' + item">
                    <app-icon icon="close_small"></app-icon>
                  </button>
                </mat-chip-row>
              </mat-chip-grid>
              <input
                placeholder="Search candidates"
                #candidateAutoCompleteInput
                [formControl]="applicationAcControl"
                [matChipInputFor]="candidateChipGrid"
                [matAutocomplete]="candidateAuto"
              />
              <mat-autocomplete
                [displayWith]="displayFullNameFn"
                #candidateAuto="matAutocomplete"
                (optionSelected)="candidateSelected($event)"
              >
                <mat-option
                  *ngFor="let application of applicationAc$ | async"
                  [value]="application"
                >
                  <!-- <p *ngIf="!checkIfDuplicate(panel)">
                    {{ panel.employeeName }}
                  </p> -->
                  {{ application.name }} ({{ application.mobile }})
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-inner">
            <label class="form-label" for="employeeId">
              Panel member<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select
                (selectionChange)="onSelect($event, true)"
                placeholder="Select panel member"
                name="employeeId"
                multiple="false"
                formControlName="employeeId"
              >
                <mat-option
                  [title]="employee.emailId"
                  [value]="employee.employeeId"
                  *ngFor="let employee of panelMembers"
                  >{{ employee.employeeName }}</mat-option
                >
              </mat-select> -->

              <input
                [formControl]="primaryPanelAcControl"
                type="text"
                placeholder="search primary panel member"
                aria-label="string"
                matInput
                [matAutocomplete]="primaryAutoComplete"
              />
              <mat-autocomplete
                [displayWith]="displayEmailFn"
                autoActiveFirstOption
                #primaryAutoComplete="matAutocomplete"
                (optionSelected)="priSelected($event)"
              >
                <mat-option
                  *ngFor="
                    let item of primaryPanelAc$ | async;
                    let index = index
                  "
                  [value]="item"
                >
                  {{ item.employeeName | titlecase }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="secondaryEmployeeIds" class="form-label"
              >Secondary panel members</label
            >
            <mat-form-field appearance="outline">
              <mat-chip-grid #chipGrid aria-label="Secondary panel selection">
                <mat-chip-row
                  *ngFor="let fruit of selectedPanel"
                  (removed)="remove(fruit.employeeId)"
                >
                  {{ fruit.employeeName }}
                  <button matChipRemove [attr.aria-label]="'remove ' + fruit">
                    <app-icon icon="close_small"></app-icon>
                  </button>
                </mat-chip-row>
              </mat-chip-grid>
              <input
                placeholder="Search secondary panel members"
                #secAutoCompleteInput
                [formControl]="secondaryPanelAcControl"
                [matChipInputFor]="chipGrid"
                [matAutocomplete]="secondaryPanelAuto"
              />
              <mat-autocomplete
                [displayWith]="displayEmailFn"
                #secondaryPanelAuto="matAutocomplete"
                (optionSelected)="secSelected($event)"
              >
                <mat-option
                  *ngFor="let panel of secondaryPanelAc$ | async"
                  [value]="panel"
                >
                  <!-- <p *ngIf="!checkIfDuplicate(panel)">
                    {{ panel.employeeName }}
                  </p> -->
                  {{ panel.employeeName }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>

            <!-- <mat-form-field>
              <mat-select
                (selectionChange)="onSelect($event, false)"
                name="secondaryEmployeeIds"
                placeholder="Select Candidates"
                formControlName="secondaryEmployeeIds"
                multiple
              >
                <mat-option
                  [title]="employee.emailId"
                  [value]="employee.employeeId"
                  *ngFor="let employee of secondaryPanelMembers"
                  >{{ employee.employeeName }}</mat-option
                >
              </mat-select>
            </mat-form-field> -->
          </div>
        </div>
