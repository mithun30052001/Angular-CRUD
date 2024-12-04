https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req-details-dialog.component.html

<div class="follow-up-modal">
  <div class="follow-up-header">
    <h5 class="heading-text">Req Details</h5>
    <div>
      <app-icon icon="close" (click)="closeDialog()"></app-icon>
    </div>
  </div>
  <div class="follow-up-body">
    <ng-container>
      <div class="followup-interview">
        <div class="folowup-table table-responsive mt-3">
          <table mat-table [dataSource]="dataSource">
            <ng-container matColumnDef="requisitionId">
              <th mat-header-cell *matHeaderCellDef>Requisition ID</th>
              <td mat-cell *matCellDef="let element">
                {{ element.requisitionId }}
              </td>
            </ng-container>

            <ng-container matColumnDef="vendor">
              <th mat-header-cell *matHeaderCellDef>Vendor</th>
              <td mat-cell *matCellDef="let element">{{ element.vendor }}</td>
            </ng-container>

            <ng-container matColumnDef="emp">
              <th mat-header-cell *matHeaderCellDef>emp</th>
              <td mat-cell *matCellDef="let element">
                {{ element.emp }}
              </td>
            </ng-container>

            <ng-container matColumnDef="direct">
              <th mat-header-cell *matHeaderCellDef>direct</th>
              <td mat-cell *matCellDef="let element">
                {{ element.direct }}
              </td>
            </ng-container>

            <ng-container matColumnDef="campus">
              <th mat-header-cell *matHeaderCellDef>campus</th>
              <td mat-cell *matCellDef="let element">{{ element.campus }}</td>
            </ng-container>

            <ng-container matColumnDef="hrEmail">
              <th mat-header-cell *matHeaderCellDef>hrEmail</th>
              <td mat-cell *matCellDef="let element">{{ element.hrEmail }}</td>
            </ng-container>

            <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
            <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
          </table>
        </div>
      </div>
    </ng-container>
  </div>
  <div class="follow-up-footer">
    <div class="row justify-content-end">
      <div class="col-lg-3 col-6">
        <button
          title="Close model"
          (click)="closeDialog()"
          class="ags-primary-btn ags-hxl56 ags-padding1624 btn-font16"
        >
          Close
        </button>
      </div>
    </div>
  </div>
</div>

exception

 <section class="mt-3">
        <div
          class="alert example-container mat-elevation-z8 text-center"
          *ngIf="dataSource.length === 0"
        >
          No Published Requisitions Found
        </div>
      </section>
