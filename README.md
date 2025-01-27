https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

application service method

getPanelMembersV2(searchKey: string, employeeIds: string[] = []) {
    return this.httpClient.post(
      `${environment.API_URL}candidate-services/panel/member/get/v2?jobType=VOICE`,
      {
        employeeIds,
        searchKey,
      }
    );
  }


panel-reschedule.ts

export class PanelRescheduleDialogComponent{
 panelLookup(value: string): Observable<any> {
      const toAvoid = this.selectedPanel.map((e: PanelMember) => e.employeeId);
  
      return this.applicationService
        .getPanelMembersV2(value?.toLowerCase(), uniq(toAvoid))
        .pipe(
          map((results: any) => results),
          catchError((_) => {
            return of(null);
          })
        );
    }
initiateform() {
if (value.employeeId) {
           this.primaryPanelAc$ = this.panelForm.get('employeeId')?.valueChanges.pipe(
               startWith(this.primaryPanelAcControl.value || ''),
               debounceTime(300),
             switchMap((value) => {
                 console.log('WHILE TYPING PRIMARY MEMBER.....');
                 if (value !== '') {
                   if (value?.employeeName) {
                     console.log('WHILE TYPING PRIMARY MEMBER NULL.....');
                     return of(null);
                   }
                    console.log('WHILE TYPING PRIMARY MEMBER NOT NULL.....');
                   return this.panelLookup(value);
                 } else {
                   return this.panelLookup('');
                 }
               })
             );
        }
}
}
