# Post-User-Confirmation Master Table Sync Template

When user says "sync item X," "item 1 can be updated," "only sync partners, don't update operational status," execute this workflow.

## Steps

1. Read the corresponding record from the Projects & Milestones table.
2. Review key fields:
   - Company
   - Target City Original / Chinese Name / Associated City
   - Source Link
   - Confirmation Status
   - Confidence Level
   - Tables to Update
   - Sync Notes
   - Platform Partner / Fleet Supplier / Local Operator
   - Current Fleet Size / Planned Fleet Size
3. Update master tables per user's confirmed scope.
4. Write back sync status and sync notes to Projects table.

## Update Order

Priority order:

1. Region Dictionary: if city doesn't exist and user confirms addition.
2. Company Info: if company doesn't exist and user confirms addition.
3. Partner Details: if partner is clearly identified.
4. Company × Region Ops Status: if company + city + status/partnership facts are clear.
5. Projects & Milestones: write back sync results.

## Notes

- When user only confirms partial fields, only update within confirmed scope.
- Plans, MOUs, exploration partnerships should not directly change operational status.
- Current fleet size and planned fleet size must be written separately.
- Leave uncertain fields blank, do not fill by guessing.
- If any step fails, stop or skip the failed item, and note in project table sync notes.

## Write-back Example

```text
Sync Status: Synced
Sync Notes: Added region "Miami"; added Waymo-Miami operational status record with status "Partnership Reached"; did not fill current fleet size as source did not disclose. Partner Uber linked to Partner Details.
```
