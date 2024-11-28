https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req-mapping-edit-dialog.ts

import { Component, Inject } from '@angular/core';
import { MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { ReqService } from '@/src/app/shared/services/req.service';
import { UserProfile } from '@/src/app/interfaces/app-interface';
import {
  FormBuilder,
  FormGroup,
  Validators,
  FormControl,
  AbstractControl,
  ValidationErrors,
} from '@angular/forms';
import { Requisition } from '../req-mapping/req-mapping.component';
import { Job } from '@/src/app/shared/services/job-posting.service';
import { ElasticSearchResponse } from '@/src/app/interfaces/app-interface';
import { ApplicationService } from '@/src/app/shared/services/job-application.service';
import {
  catchError,
  debounceTime,
  map,
  Observable,
  of,
  startWith,
  switchMap,
} from 'rxjs';
import { AuthService } from '@/src/app/shared/services/auth.service';


@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
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

  originalOpenNumbers: number | null = null;
  openNumbersChanged = false;

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
      const formattedOnboardingDate = onboardingDateObject
        ?.toISOString()
        .split('T')[0];
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
       this.originalOpenNumbers = rowData.openNumbers;
      this.reqForm.addControl(
        'openNumbers',
        new FormControl(rowData.openNumbers, Validators.required)
      );
      this.reqForm.addControl('openNumbersProof', new FormControl(null));

       this.reqForm.get('openNumbers')?.valueChanges.subscribe((newValue) => {
         this.openNumbersChanged = newValue !== this.originalOpenNumbers;
       });
    }
  }

  validateHrIdCtrl(ctrl: AbstractControl): ValidationErrors | null {
    const val = ctrl.value;
    if (!val || val === '' || !val?.id) {
      return {
        error: true,
      };
    }
    return null;
  }

  ngOnInit(): void {
    console.log('MY DIALOG DATA INIT', this.rowData);
    this.getJobs();

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

  lookup(value: string): Observable<any> {
    return this.authService.hrSearch(value?.toLowerCase()).pipe(
      map((results) => results),
      catchError((_) => {
        return of(null);
      })
    );
  }

  displayFullNameFn(option: UserProfile | null) {
    console.log('Option', option);
    return option ? option?.fullName ?? option?.email?.split('@')[0] : '';
  }

  transformList(response: ElasticSearchResponse<any>) {
    return response.hits.hits.map((job) => {
      return job._source;
    });
  }

  getJobs() {
    this.applicationService
      .getAllJobs({
        orderBy: 'desc',
        limit: 500,
        offset: 0,
      })
      .then((res) => {
        this.jobs = this.transformList(res);
        console.log('MY JOBS', this.jobs);
      });
  }

  closeModal() {
    this.dialogRef.close();
  }

  onSubmit() {
    if (this.reqForm.valid) {
      const formValues = this.reqForm.value;
      console.log('Manager', formValues);
      formValues.manager = formValues.manager.id;
      formValues.spoc = formValues.spoc.id;
      console.log(formValues);
      if (formValues.openNumbers) {
        console.log('OpenNumber', formValues.openNumbers);
        const proofFile = this.reqForm.get('openNumbersProof')?.value;
        console.log("Proof file",proofFile)
         const requestData = {
           jobRequisition: formValues,
           file: proofFile,
        };
        
         this.reqService.updatePublishedRequisitions(requestData).subscribe(
           (response) => {
             console.log(
               'Published requisition updated successfully:',
               response
             );
             this.dialogRef.close(true);
           },
           (error) => {
             console.error('Error updating published requisition:', error);
           }
        );
        
      } else {
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
  }
}

req-mapping.component.ts
import { Component, OnInit } from '@angular/core';
import { MatIconModule } from '@angular/material/icon';
import { MatTableModule } from '@angular/material/table';
import { SharedModule } from '../../../shared/shared.module';
import { MatDialog } from '@angular/material/dialog';
import { ReqMappingEditDialogComponent } from '@/src//app/hr/components/req-mapping-edit-dialog/req-mapping-edit-dialog.component';
import { ReqUploadDialogComponent } from '../req-upload-dialog/req-upload-dialog.component';
import { ReqService } from '@/src/app/shared/services/req.service';
import { MatCardModule } from '@angular/material/card';
import { MediaMatcher } from '@angular/cdk/layout';
import { Subject } from 'rxjs';
import { get } from 'lodash-es';
import { Filters, MatPageEvent } from '@/src/app/interfaces/app-interface';
import { ListFiltersComponent } from '@/src/app/shared/components/list-filters/list-filters.component';

export interface PeriodicElement {
  sjoined: string;
  syto: string;
  sspoc: string;
  opennumber: string;
  shift: string;
  wfm: string;
  openno: string;
  action: string;
}

export interface Requisition {
  id: string;
  createdDate: string;
  updatedDate: string;
  active: boolean;
  requestType: string;
  requisitionId: string;
  clientName: string;
  openNumbers: number;
  intentDate: string;
  closureDate: string | null;
  process: string;
  location: string;
  speciality: string;
  jobType: string;
  shift: string;
  requisitionStatus: string;
  spoc: string | null;
  split: string;
  role: string;
  onboardingDate: string | null;
  published: boolean;
}

const ELEMENT_DATA: PeriodicElement[] = [
  {
    sjoined: '20',
    syto: '20',
    sspoc: '20',
    opennumber: '20',
    shift: '20',
    wfm: 'eee',
    openno: 'eeee',
    action: '',
  },
];
@Component({
  selector: 'app-req-mapping',
  templateUrl: './req-mapping.component.html',
  styleUrls: ['./req-mapping.component.scss'],
  standalone: true,
  imports: [MatTableModule, MatIconModule, SharedModule, MatCardModule],
})
export class ReqMappingComponent
  extends ListFiltersComponent
  implements OnInit
{
  // mobileView: any;
  requisitions: Requisition[] = [];
  filters: Filters = {};
  totalRecords = 0;

  constructor(
    media: MediaMatcher,
    public dialog: MatDialog,
    private reqService: ReqService
  ) {
    super({ userType: 'string' }, media);
  }
  private searchSubject = new Subject<string>();
  displayedColumns = [
    'requestType',
    'requisitionId',
    'process',
    'clientName',
    'speciality',
    'jobType',
    'location',
    'shift',
    'openNumbers',
    'action',
  ];

  dataSource = ELEMENT_DATA;
  queryParams: any;

  ngOnInit(): void {
    this.getJobRequisitions();
  }

  getJobRequisitions() {
    this.reqService.getRequisitions().subscribe({
      next: (data: Requisition[]) => {
        this.requisitions = data;
        this.totalRecords = get(data, 'hits.total.value', 0);
      },
      error: (err) => {
        console.error('Error fetching requisitions', err);
      },
    });
  }
  onPageChange(event: MatPageEvent) {
    this.onPagination(event);
    this.getJobRequisitions();
  }
  
  onSearching(searchString: string) {
    this.searchSubject.next(searchString);
  }

  reqMappingEdit(rowData: Requisition) {
    console.log('amhsdbhasbdhasbdfb',rowData);
    this.dialog.open(ReqMappingEditDialogComponent, {
      maxWidth: this.mobileView ? '500px' : '100%',
      width: this.mobileView ? '100%' : '50%',
      height: 'auto',
      ...(this.mobileView && {
        position: { bottom: '5px' },
      }),
      closeOnNavigation: true,
      disableClose: false,
      data: rowData,
    });
  }

  reqMappingUpload() {
    this.dialog.open(ReqUploadDialogComponent, {
      maxWidth: this.mobileView ? '700px' : '100%',
      width: this.mobileView ? '100%' : '40%',
      height: 'auto',
      ...(this.mobileView && {
        position: { bottom: '5px' },
      }),
      closeOnNavigation: true,
      disableClose: false,
    });
  }
}
