https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

profile-html

<div class="row d-flex align-items-center">
  <div class="col-sm-4 col-lg-2 col-3">
    <div class="profile-avator position-relative">
      <!-- Profile Image -->
      <img src="assets/images/profile-pic.png" alt="" class="img-fluid" />
      
      <!-- Edit Icon placed at the bottom-left of the image -->
      <button class="icon-button position-absolute bottom-0 start-0 mb-2 ms-2" (click)="openFileSelector()">
        <app-icon icon="edit_ic-pencil-icon" title="Edit Profile Picture" />
      </button>
      
      <!-- File Input (Hidden) -->
      <input type="file" #fileInput class="d-none" (change)="onFileSelected($event)" accept="image/*" />
    </div>
  </div>
  <div class="col-sm-7 col-lg-9 col-8">
    <div class="profile-left-heading">
      <p class="">{{ profile.fullName }}</p>
      <p class="">{{ profile.mobile }}</p>
      <p class="mail-color">{{ profile.email }}</p>
    </div>
  </div>
  <div class="col-sm-1 col-lg-1 col-1 d-flex justify-content-end">
    <div class="profile-heading-edit">
      <button class="icon-button" (click)="quickEdit()">
        <app-icon icon="edit_ic" title="Quick edit" />
      </button>
    </div>
  </div>
</div>



profile.ts


import { Component, ViewChild, ElementRef } from '@angular/core';

@Component({
  selector: 'app-profile',
  templateUrl: './profile.component.html',
  styleUrls: ['./profile.component.scss'],
})
export class ProfileComponent {
  @ViewChild('fileInput') fileInput: ElementRef;

  // Method to trigger file selector
  openFileSelector() {
    this.fileInput.nativeElement.click(); // Programmatically click the hidden file input
  }

  // Method to handle file selection
  onFileSelected(event: any) {
    const file = event.target.files[0];
    if (file) {
      // Handle the file, e.g., upload it to the server or preview it
      console.log('Selected file:', file);
      this.uploadProfileImage(file);
    }
  }

  // Method to upload the selected image (stub for actual file upload logic)
  uploadProfileImage(file: File) {
    // You would typically send the file to a server here
    console.log('Uploading file:', file);
  }
}



prfile.scss

.profile-avator {
  position: relative;
  width: 100%;
  height: auto;
}

.icon-button {
  background: transparent;
  border: none;
  padding: 0;
  cursor: pointer;
}

.icon-button app-icon {
  width: 30px;  /* Adjust the icon size */
  height: 30px;
  color: white;  /* Icon color */
}

.position-absolute {
  position: absolute;
}

.bottom-0 {
  bottom: 0;
}

.start-0 {
  left: 0;
}

.mb-2 {
  margin-bottom: 10px;
}

.ms-2 {
  margin-left: 10px;
}

.d-none {
  display: none;
}
