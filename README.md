https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req.service.ts

import { environment } from '@/environments/environment';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class ReqService {

  constructor(private http: HttpClient) {}

  // Method to upload requisition details
  uploadReqdetails(file: File): Observable<any> {
    const formData = new FormData();
    formData.append('file', file, file.name);

    // API endpoint for uploading the file
    const url = `${environment.API_URL}requisition/upload`;

    return this.http.post<any>(url, formData);
  }
}



requpload.component.ts

import { Component } from '@angular/core';
import { MatDialog, MatDialogRef } from '@angular/material/dialog';
import { ReqService } from './req.service';  // Import the ReqService
import { catchError, of } from 'rxjs';  // To handle errors

@Component({
  selector: 'app-req-upload-dialog',
  templateUrl: './req-upload-dialog.component.html',
  styleUrls: ['./req-upload-dialog.component.scss'],
})
export class ReqUploadDialogComponent {
  selectedFile: File | null = null;
  fileError: string | null = null;
  uploadSuccess: boolean = false;
  uploadError: string | null = null;

  constructor(
    private dialog: MatDialog,
    public dialogRef: MatDialogRef<ReqUploadDialogComponent>,
    private reqService: ReqService  // Inject the ReqService
  ) {}

  closeModal() {
    this.dialogRef.close();
  }

  // File selection handler
  onFileSelected(event: Event): void {
    const input = event.target as HTMLInputElement;
    if (input?.files?.length) {
      const file = input.files[0];
      if (file.type !== 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet') {
        this.fileError = 'Please upload a valid .xlsx file.';
        this.selectedFile = null;
      } else {
        this.fileError = null;
        this.selectedFile = file;
      }
    }
  }

  // Form submission handler
  onSubmit(): void {
    if (this.selectedFile) {
      this.uploadReqdetails(this.selectedFile);
    }
  }

  // Upload file method that calls the service
  uploadReqdetails(file: File): void {
    this.reqService.uploadReqdetails(file).pipe(
      catchError((error: HttpErrorResponse) => {
        // Handle errors during the upload
        this.uploadError = `Upload failed: ${error.message}`;
        this.uploadSuccess = false;
        return of(null);  // Return an observable with null value to continue the flow
      })
    ).subscribe(response => {
      if (response) {
        // Assuming the response indicates success
        this.uploadSuccess = true;
        this.uploadError = null;
        console.log('File uploaded successfully:', response);
      }
    });
  }
}
