# Coach client manager: acceptance criteria

**Draft: Kidd to review and own.** Each needs at least one automated test.

1. A coach in business A can't read or change any row of business B through direct database calls.
2. A coach can't read clients who aren't assigned to them, including their health notes and progress photos.
3. A client can read only their own rows.
4. Booking two overlapping sessions for the same coach fails; so does for the same client.
5. Booking without an active package with a balance fails with "no sessions left" or "package expired".
6. Two simultaneous bookings for a package with one session left: exactly one succeeds. *(Database test with two connections.)*
7. Cancelling 13 hours before a session (12-hour window) returns the session to the balance; cancelling 11 hours before doesn't.
8. The package balance always equals sessions total minus counted sessions, and never goes below zero.
9. A second check-in for the same client and week fails.
10. A client can edit a check-in before it's reviewed and not after.
11. Assigning a template copies it; editing the template afterwards doesn't change the client's programme.
12. A client can't have two active programmes.
13. Changing a package's price doesn't change the price recorded on packages already sold.
14. The weight trend chart's 7-day average matches a hand-calculated value for the sample data.
15. A client who missed two check-ins in a row appears in "needs attention".
16. Each demo account signs in from the portfolio page with one click. *(End-to-end.)*
17. A client can complete a logged workout on a phone-sized screen. *(End-to-end at 390 px width.)*
