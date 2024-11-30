https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

ProfileUpload.service.ts

// profile.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class ProfileService {
  // Store the profile image URL or any relevant profile data
  private profileImageSource = new BehaviorSubject<string>('');
  profileImage$ = this.profileImageSource.asObservable();

  constructor() {}

  // Method to update the profile image
  updateProfileImage(imageUrl: string): void {
    this.profileImageSource.next(imageUrl); // Emit the new image URL
  }

  // Optionally, you can have a method to fetch the profile data from the server.
  getProfileImage() {
    return this.profileImageSource.getValue(); // Return current image URL (for initial load)
  }
}

header.component.ts

// header.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
import { ProfileService } from 'path-to-your-service/profile.service';
import { Subscription } from 'rxjs';

@Component({
  selector: 'app-header',
  templateUrl: './header.component.html',
})
export class HeaderComponent implements OnInit, OnDestroy {
  profileImage: string = '';
  private profileImageSubscription: Subscription;

  constructor(private profileService: ProfileService) {}

  ngOnInit(): void {
    // Subscribe to profile image updates
    this.profileImageSubscription = this.profileService.profileImage$.subscribe(
      (imageUrl) => {
        this.profileImage = imageUrl;
      }
    );

    // Optionally, set initial image on load
    this.profileImage = this.profileService.getProfileImage();
  }

  ngOnDestroy(): void {
    // Unsubscribe to avoid memory leaks
    if (this.profileImageSubscription) {
      this.profileImageSubscription.unsubscribe();
    }
  }
}
