populateFormWithEventData() {
    if (this.eventData) {
      const { panelMembers, candidates, start, end, timezone, job } = this.eventData;
      
      // Set date and timing
      this.panelForm.patchValue({
        date: moment(start).format('YYYY-MM-DD'),
        timings: {
          startTime: moment(start).format('HH:mm'),
          endTime: moment(end).format('HH:mm')
        },
        timezone: timezone,
      });

      // Fill panel members
      const primaryMember = panelMembers.find((member: any) => member.primaryPanelMember === true);
      const secondaryMembers = panelMembers.filter((member: any) => member.primaryPanelMember === false);

      this.primaryPanelAcControl.setValue(primaryMember);
      this.secondaryPanelAcControl.setValue(secondaryMembers.map((member: any) => member.employeeId));

      // Fill candidates
      if (candidates && candidates.length > 0) {
        this.selectedCandidates = candidates;
        this.applicationAcControl.setValue(candidates.map((candidate: any) => candidate.applicationId));
      }

      // Fill job data
      if (job && job.jobId) {
        this.jobAcControl.setValue(job);
        this.panelForm.patchValue({ jobId: job });
      }
    }
  }
