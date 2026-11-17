# Libby backup gate

Run before the first Libby tag or untag in a session. Tags are easy to lose and **cannot be restored from an export**, so the user must understand the model, pick a workflow, and confirm a backup. Ask with AskUserQuestion, one step at a time, and offer to guide each step.

## What to tell the user (from Libby Help)

- Libby has no account. **Tags and the timeline live on each device.** Loans, holds, borrowing history and reading progress belong to the library card. (help.libbyapp.com/en-us/6004.htm)
- Tags **sync only between devices linked** with a recovery passkey or a setup code. A phone, tablet or browser where the user just added their card is a separate island: it shows loans and holds, not tags. (6023.htm)
- Libby Help calls the device whose tags you keep the **main device**. Setup codes copy cards, loans, holds and tags **from** the device where you type the code **to** the device showing it. (6070.htm, 6137.htm)
- **Reset Everything wipes that device's tags.** Recovering onto a device that already has data means resetting it first. Clearing browser site data or uninstalling the app should be treated the same way. (6137.htm, 6004.htm)
- The timeline never syncs. Deleted tags cannot be recovered. (6052.htm, 6090.htm)

## Gate steps

1. **Where are the tags now?** Ask which device holds the tags they care about. That is the main device. Save it to `libby_main_device`.
2. **Back up.** On the main device: create a **recovery passkey** (Libby's recommended backup), and **export every tag** (Tags > tag > Actions > Export Tag > Spreadsheet) and the timeline (Shelf > Timeline > Actions > Export Timeline). Exports are a record to rebuild from by hand, not a restore; there is no import. Wait for the user to confirm; save `libby_backup_confirmed_on`.
3. **Choose a workflow** (both explained; recommend A):
   - **A. Linked devices (default).** Link the browser to the main device once: in the browser choose "Display Setup Code", on the main device enter it at Menu > Copy To Another Device (or recover with the passkey). From then on, tags made in the browser sync to every linked device. Check by tagging one test book and seeing it on the phone.
   - **B. Browser workspace, then transfer (fallback for unlinked setups).** Copy the main device into the browser (setup code as above), do all tagging in the browser, then make the browser the source: on each target device, **export its tags first**, Reset Everything, and recover from the browser with a setup code. Warn plainly: the reset deletes that device's own tags, and anything tagged there since the copy is lost unless exported and re-added.
4. **Confirm the browser is the right instance.** Open `libbyapp.com/tags` in the browser the skill will drive and compare tag names and counts with the main device. If they differ, stop and resolve before tagging.
5. Proceed to [libby-actions.md](libby-actions.md).

## When the user wants to skip the gate

Allow it once they have heard the reset warning in one sentence, and note in the run report that no backup was confirmed. Never run Reset Everything or a device transfer yourself; guide the user through it.
