https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

viewRemarkImage(remarkId: string): void {
    this.reqService.getRemarkImage(remarkId).subscribe(
      (imageData) => {
        // Handle image display logic, for example, open in a new window or modal
        const imageUrl = URL.createObjectURL(imageData);
        window.open(imageUrl, '_blank');
      },
      (error) => {
        console.error('Error fetching remark image:', error);
      }
    );
  }
