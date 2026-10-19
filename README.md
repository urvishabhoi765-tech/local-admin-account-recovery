# Local Administrator Sign-in Failure

A hands-on Windows 11 troubleshooting lab. A user ticket reports that the built-in local Administrator account cannot sign in. I investigated it as a standard user, found the real cause, and fixed it **without resetting the password**.

**Author:** Urvisha Bhoi
**Lab date:** October 18, 2026

## The ticket

> "I can't sign in to WIN11-WS01 with the local Administrator account."

**Cause:** the built-in Administrator account was **disabled**. The password was correct the whole time.

## Files

| File | Description |
|---|---|
| `Local_Administrator_Troubleshooting_Report` | Full report: every step and command explained, with 18 screenshots |
| `README.md` | This file |

## Lab environment

| Item | Value |
|---|---|
| Computer | WIN11-WS01, Windows 11 Pro, joined to testuser.local |
| Account with the problem | Administrator (built-in local account) |
| Standard domain user | IT User02 (`TESTUSER\ituser02`) |
| Backup local admin | SupportAdmin (member of local Administrators) |

## How it was done

### 1. Set up the lab
- Created a backup local admin: `net user SupportAdmin * /add`, `net localgroup Administrators SupportAdmin /add`
- Enabled the built-in Administrator and set a password. The first attempt failed with **error 2245**, because the account had no password and the domain password policy applies.
- Disabled it again with `net user Administrator /active:no` to plant the problem.

### 2. Investigate
| Step | Tool / command | Finding |
|---|---|---|
| Sign in as `.\Administrator` | Sign-in screen | "Your account has been disabled" |
| Check the account as IT User02 | MMC > Local Users and Groups | **Account is disabled** is ticked |
| Try to fix it as IT User02 | Untick > Apply | **Access is denied** |
| Why? | `whoami /groups` | IT User02 is not in BUILTIN\Administrators |
| Check the backup admin | Member Of tab, `net localgroup Administrators` | SupportAdmin is a local admin |

### 3. Fix
- Signed in as `.\SupportAdmin`, opened `lusrmgr.msc`, and cleared **Account is disabled** on Administrator.
- **Did not reset the password**, so only one thing changed.

### 4. Verify
- Signed in as `.\Administrator` with the **original password**: it worked.
- This proves the disabled state, not the password, was the cause.

### 5. Password reset: reviewed, not used
Windows warns that an admin password reset can make the account lose access to **encrypted files, stored passwords and certificates**. Because the original password worked, no reset was needed.

## Key lessons
- A sign-in failure does not always mean a wrong password.
- `.\` means a local account; `TESTUSER\` means a domain account.
- A standard user can view account settings but cannot change them.
- An enabled account is not automatically an administrator; check group membership.
- Change one thing at a time to prove the cause.
- Avoid admin password resets unless truly necessary.

## Security follow-up
- Keep the built-in Administrator disabled when not needed; disable it again after troubleshooting.
- Use a named admin account (like SupportAdmin) for maintenance.
- Manage local admin passwords with Windows LAPS.
- Review local Administrators group membership regularly.

## Skills demonstrated
Local account management, Local Users and Groups (MMC / lusrmgr.msc), `net user`, `net localgroup`, `whoami`, Windows permissions, structured troubleshooting, technical documentation
