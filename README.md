https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

service.ts

pdatePublishedRequisitions(data: {
    jobRequisition: any;
    file?: File;
  }): Observable<any> {
    const formData = new FormData();
    formData.append('jobRequisition', JSON.stringify(data.jobRequisition));

    if (data.file) {
      formData.append('file', data.file, data.file.name);
    }

    const url = `${environment.API_URL}job-services/requisition/job/update`;
    return this.http.post<any>(url, formData);
  }

onSubmit() {
  if (this.reqForm.valid) {
    const formValues = this.reqForm.value;
    console.log('Manager', formValues.manager);
    formValues.manager = formValues.manager.id;
    formValues.spoc = formValues.spoc.id;
    console.log(formValues);

    if (formValues.openNumbers) {
      console.log('OpenNumber', formValues.openNumbers);
      
      // Get the file from form control
      const proofFile = this.reqForm.get('openNumbersProof')?.value;

      // Prepare data object with form values and file
      const requestData = {
        jobRequisition: formValues,
        file: proofFile,
      };

      // Call the updatePublishedRequisitions method
      this.reqService.updatePublishedRequisitions(requestData).subscribe(
        (response) => {
          console.log('Published requisition updated successfully:', response);
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
