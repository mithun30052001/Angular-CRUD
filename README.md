populateFormWithEventData(eventData: any): void {
  // Set the Job ID (Job Title)
  this.panelForm.patchValue({
    jobId: {
      id: eventData.jobId, 
      jobTitle: eventData.title, 
      oracleJobReference: eventData.oracleJobReference,
      location: eventData.location,
      display: eventData.display,
    }
  });

  // Set the Requisition ID
  this.panelForm.patchValue({
    requisitonId: eventData.id
  });

  // Set Candidates (you may need to extract candidate IDs, emails, and names)
  const candidateIds = eventData.candidates.map(c => c.candidateId);
  this.panelForm.patchValue({
    applicationIds: candidateIds
  });

  // Set Primary Panel Member
  const primaryPanelMember = eventData.panelMembers.find(pm => pm.primaryPanelMember === true);
  if (primaryPanelMember) {
    this.panelForm.patchValue({
      primaryPanelAcControl: primaryPanelMember
    });
  }

  // Set Secondary Panel Members
  const secondaryPanelMembers = eventData.panelMembers.filter(pm => pm.primaryPanelMember === false);
  this.panelForm.patchValue({
    secondaryPanelAcControl: secondaryPanelMembers
  });

  // Set Date and Time
  const startDate = new Date(eventData.start);
  this.panelForm.patchValue({
    date: startDate,
    timings: {
      startTime: new Date(eventData.start),
      endTime: new Date(eventData.end)
    }
  });

  // Set Timezone
  this.panelForm.patchValue({
    timezone: {
      name: eventData.timezone.name,
      abbreviation: eventData.timezone.abbreviation,
      mainCities: eventData.timezone.mainCities
    }
  });
}
