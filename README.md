Overview
========

This package adds the ability to sync org-mode tasks with
Toodledo, a powerful web-based todo list manager that welcomes 3rd
party integrations.  (See http://www.toodledo.com/)

This version of `org-toodledo' utilizes version 2.0 of the Toodledo API. 

Synchronizing For The First Time
================================

The first step in using org-toodledo is to initialize a file and
synchronize tasks.  Simply create a new file, change the mode to
`org-mode', then call `org-toodledo-initialize'.  This will create
a new heading called "TASKS" (by default) and will import all
non-deleted tasks from Toodledo as sub-headings beneath "TASKS".

If you already have an existing list of tasks in org file, open the
org file first.  Move the cursor to the headling where you want
imported tasks from Toodledo to be inserted into the buffer.  Call
`org-toodledo-initialize'.  This will import all tasks from the
server as well as pushing existing tasks in the org file back to
the server.

Once an org-file has been initialized, the heading selected will
be given a few Toodledo specific properties that are used to track
the status of synchronization:

    * TASKS 
      :PROPERTIES:
      :ToodledoLastSync: 1315343842
      :ToodledoLastEdit: 1315337478
      :ToodledoLastDelete: 1314972230
      :OrgToodledoVersion: 2.3
      :END:

This is referred to as the 'base Toodledo entry'.

Synchronizing Tasks
===================

The local org-file can be synchronized with the server at any time
by calling `org-toodledo-sync'.  When called, the following steps
are performed:

  1. Tasks added to the server since the last sync are downloaded
     and inserted as sub-headings to the Toodledo base heading (has
     the `ToodledoLastSync' property)

  2. Tasks modified on the server are compared against the local
     copy.  If the local copy was not modified since the last sync,
     the local copy is updated.  If local copy was modified, the
     server copy is inserted *after* the local copy as a duplicate.
     The user must manually merge any changes

  3. Tasks deleted on the server are removed entirely from the
     local org file.

  4. Tasks modified locally are pushed to the server as edits.

  5. Tasks created and not yet prseent on the server are pushed as
     new tasks.

  6. Tasks marked for deletion are deleted from the server, and
     then purged from the local file.

Changes to tasks are automatically detected by computing a hash of
the task fields.  This hash is computed and saved as a property of
the task on sync.  When the next sync occurs, the hash value is
compared and if it differs, the task is considered modified.  This
eliminates the need for the user to mark tasks as modified or
remembere which tasks have changed -- it's all automatic!

Note that `org-toodledo-sync' scans the entire file for tasks, not
just subheadings of the base entry.

Adding New Tasks
================

To add a new task on the server, just create a new headline
anywhere in the org file and give the headline a TODO keyword.
When ready, call `org-toodledo-sync' to push new tasks to the
server.

Deleting Tasks
==============

Tasks cannot simply be killed from the org-file like text if the
were already synced with the server since they will just come back
the next time `org-toodledo-sync' is called.  Instead, they must be
marked as deleted by calling `org-toodledo-mark-task-deleted'.  Call
this function from any point within the task.  At the next sync, 
the task will be deleted from the server and then killed from the 
local file.

Note that it may not be necessary to delete tasks in this way.  Instead
complete the task and let Toodledo archive completed tasks.

Toodledo Fields
===============

The table lists the possible Toodledo fields and how they are
mapped to org-mode style tasks:

<table>
<tr><td> Toodledo Field </td><td> Org-mode               </td><td> Comments                                     </td></tr>
<tr><td> id             </td><td> Property :ToodledoID:  </td><td> If present, this task was previoiusly synced </td></tr>
<tr><td> title          </td><td> Heading                </td><td> Heading minus TODO state, priority and tags  </td></tr>
<tr><td> status         </td><td> TODO state             </td><td> See `org-toodledo-status-to-org-map'         </td></tr>
<tr><td> startdate      </td><td> SCHEDULED              </td><td> startdate/startime are GMT                   </td></tr>
<tr><td> starttime      </td><td> SCHEDULED              </td><td>                                              </td></tr>
<tr><td> duedate        </td><td> DEADLINE               </td><td> duedate/duetime are GMT                      </td></tr>
<tr><td> duetime        </td><td> DEADLINE               </td><td>                                              </td></tr>
<tr><td> completed      </td><td> CLOSED                 </td><td> Timestamp when the task was marked completed </td></tr>
<tr><td> repeat         </td><td> Repeat interval        </td><td>                                              </td></tr>
<tr><td> repeatfrom     </td><td>                        </td><td>                                              </td></tr>
<tr><td> context        </td><td> Tag                    </td><td> Context string "Work" becomes a tag :@Work:  </td></tr>
<tr><td> folder         </td><td> Prop :ToodledoFolder:  </td><td> Only used if not using headings for folders, see below  </td></tr>
<tr><td> goal           </td><td> Prop :ToodledoGoal:    </td><td>                                              </td></tr>
<tr><td> priority       </td><td> Priority               </td><td> 3=>A, 2=>B, 1=>C, -1,0 => D                  </td></tr>
<tr><td> note           </td><td> Body                   </td><td> Body of the task minus the properties        </td></tr>
<tr><td> length         </td><td> Effort                 </td><td>                                              </td></tr>
<tr><td> parent         </td><td>                        </td><td> Links tasks parent/child                     </td></tr>
<tr><td> tag            </td><td> Tag                    </td><td> org-mode tags, note context is also a tag    </td></tr>
</table>

TODO States
===========

The TODO states from Toodledo are mapped to org-mode states via the
`org-toodledo-status-to-org-map' alist.   This can be customized to
choose your own TODO states, but all 10 states from Toodledo should
be mapped, even if only a subset are used in org-mode.

In order to cycle through all the states recognized by Toodledo,
put a line like the following somewhere in your org file:

    #+SEQ_TODO: TODO(t) DELEGATED(g) SOMEDAY(s) WAITING(w) | DONE(d) CANCELLED(c) REFERENCE(r) 

Contexts
========

Toodledo 'Contexts' allow you to split tasks into contexts such as
Work and Home.  Contexts are mapped to org tags with the '@' keyword,
:@Work: and :@Home:.

Currently only contexts already on the server are recognized.  Setting
the task context of :@Phone: when Phone is not a valid context will 
loose the context.

Subtasks
========

Sub-tasks are supported by Toodledo with a Pro account subscription.  
When enabled, a 2-level task hierarchy is supported:

    * TODO Write a best-selling novel
    ** DONE Make an outline
    ** WAITING Call Susan about the contract
    ** TODO Finish writing
    ** TODO Profit

The parent/child relationship is tracked dynamically at the time
of sync, looking for the next heading up for each task, and if present
and a task, link the task to the parent.

Bi-directional synchronization is fully supported.

If the account is not a Pro account, subtasks will still be synced
to the server, but the parent/child relationship is not.  This
yields a flat list of tasks on the server.  Note that the hierarchy
in the org file is still maintained even though not on the server.

NOTE: A hierarchy of TODO items of more than 2 levels is not supported
by the server.  If 3 or more levels is present, all children will
appear directly beneath the top-most TODO item:

org-mode:  

    * TODO Level 1 item
    ** WAITING Level 1.1 item
    *** DONE Level 1.1.1 item
    ** DONE Level 1.2 item
    *** DONE Level 1.2.1 item

server:
    * TODO Level 1 item
    ** WAITING Level 1.1 item
    ** DONE Level 1.1.1 item
    ** DONE Level 1.2 item
    ** DONE Level 1.2.1 item

Note that the hierarchy is preserved in the org-mode file, it just
displays with the children flattened on the server.

Folders
=======

Folders are supported in two modes based on the variable
`org-toodledo-folder-support-mode`.  If nil, basic mode
is used and the folder associated with a task is stored
by name in the 'ToodledoFolder' property.

If set to 'heading, the folders represent headings.  In 
this mode, the folder is implicitly defined by moving
up the outline tree to the first non-TODO item.  

For example:

    * TASKS
    ** TODO Non-folder tasks
    * Folder 1
    ** TODO Task 1.1
    ** TODO Task 1.2
    * Folder 2
    ** TODO Task 2.1
    ** TODO Task 2.2

All subtasks are assigned to the same folder.  Moving a task
from one folder to another will change the folder as well. 
The heading that represents the folder will get assigned a
property 'ToodledoFolderID' that is the ID assigned by the 
server for this folder.  

Since folder association is determined by hierarchy, the
property 'ToodledoFolder' is no longer needed on a per task
basis.  

If you currently have an org-toodledo buffer with tasks and
folders using basic mode (`org-toodledo-folder-support-mode` 
set to nil), you can use the function:

    M-x org-toodledo-switch-folder-support-mode-to-headings

This will switch your setting over to 'heading and will 
automatically move around TODO subtrees according to their
assigned folder.  (This function has been tested, but I 
suggest you backup your work before running it just in case...)

Folder name changes are not yet handled.

Miscellaneous Notes
===================

 - Doesn't do lots of error trapping. Might be a good idea to
   version-control your Org file.

 - Verify handling of other tags that are not context
 
 - The body of a task is stored as the Toodledo note.  May get
   confused by asterisks, so don't use any starting asterisks in
   your body text.  (or anything that looks like an Org headline).

 - w3mexcerpt.el inlcudes things needed things from w3m (since w3m
   requires things which require things which require things which
   require an executable which is no longer readily
   available.). (sachac)

 - By default, save will ask to sync with Toodledo.  This can
   behavior can be changed via `org-toodledo-sync-on-save'.

Known Issues
============

- Attempting the following the change will fail:

        * Task 1                 * Task 1
        * Task 2       ==>       ** Task 2
        ** Task 3                ** Task 3

  The problem is that Task 2 is changed to a child before Task 3's
  parent is changed, so the server rejects it because you can't 
  can't have a 3-level heirarchy.

Future Work
===========

TODO Feature Requests: highest priority at top

[ ] access to toodledo via proxy would also be good for those
    inside proxy based firewalls. (stophlong)

[ ] Add a 'purge-completed-tasks' function -- once these tasks have
    been synced to the server, kill them locally (since they are
    backed up on toodledo).  Alternatively, move them to an archive
    file.  (cjwhite)

[ ] Option to restrict synchronization to just sync tasks under the
    the base Toodledo entry.  (cjwhite)

[ ] Support tasks across all agenda files.  (cjwhite)

Installation
============

1. Required emacs packages:
     * `w3m' or `w3mexcerpt' -- see Notes below
     * `http-post-simple' -- http://www.emacswiki.org/emacs/http-post-simple.el

2. Put this file in your load path, byte compile the file for best
   performance, see `byte-compile-file'.

3. Put the following in your .emacs:

   (push "<path-to-this-file>" load-path)
   (require 'org-toodledo)
   (setq org-toodledo-userid "<toodledo-userid>")      << *NOT* your email!
   (setq org-toodledo-password "<toodled-password>")

   ;; Useful key bindings for org-mode
   (add-hook 'org-mode-hook
          (lambda ()
            (local-unset-key "\C-o")
            (local-set-key "\C-od" 'org-toodledo-mark-task-deleted)
            (local-set-key "\C-os" 'org-toodledo-sync)
            )
          )
   (add-hook 'org-agenda-mode-hook
          (lambda ()
            (local-unset-key "\C-o")
            (local-set-key "\C-od" 'org-toodledo-agenda-mark-task-deleted)
            )
          )

4. Install 2 patches for url-http.el (these are written for 23.3, but may
   work for other versions, if not patch manually, as the diffs are
   not that complex)

   url-http.el.emacs-23.3.patch 
      - addresses http://debbugs.gnu.org/cgi/bugreport.cgi?bug=9592, 
        involving attempted connection reuse
      - addresses http://debbugs.gnu.org/cgi/bugreport.cgi?bug=8931,
        problem when sending a request with no data
        
   url-http.el.emacs-23.3.patch2 
      - addresses http://debbugs.gnu.org/cgi/bugreport.cgi?bug=10768
        fixes a problem with responses that are barely longer than 1
        TCP data packet (about 1200 bytes)
   
   To install the patches:
      $ cd $emacs_install_dir/lisp/url
      $ patch < $path_to_patch/url-http.el.emacs-23.3.patch
      $ patch < $path_to_patch/url-http.el.emacs-23.3.patch2

   Then in emacs:
      M-x byte-compile-file $emacs_install_dir/lisp/url/url-http.el

   This patch seems to apply cleanly to 23.2 as well, but is not tested there.

Change Log:
===========

2011-09-07  (cjwhite)
- First release for general distribution based on API 2.0

2011-09-18  (cjwhite)
- (Bug fix) Properly create contexts that are missing on the server. (cjwhite)
- (Bug fix) Eliminate hyphens in the tag/properties that are saved (cjwhite)
- Implemented sub-tasks -- requires pro account subscription (cjwhite)
- Added customization variable `org-toodledo-sync-import-new-tasks'

2011-09-24  (cjwhite)
- Use https if pro subscription and patch installed (url-http appears broken
  for POSTs with data and https, at least on my box).  To enable, apply the 
  patch as follows:

        $ cd $emacs_install_dir/lisp/url
        $ patch < $path_to_patch/url-http.el.emacs-23.3.patch

  Then in emacs:

        M-x byte-compile-file $emacs_install_dir/lisp/url/url-http.el

  This patch seems to apply cleanly to 23.2 as well, but is not tested there.
  Search below for "bugreport" for more details and bug references.

- Added `org-toodledo-run-tests' to load and run tests in org-toodledo-test.el.
  This uses the active account, creating/modifying/deleting tasks with the
  prefix 'ORGTOODLEDOTEST'.  All other tasks are ignored, so it *should* operate
  cleanly on an active toodledo account with multiple tasks.  If you run
  this and it does not pass all tests, please let me know  (cjwhite)
  
2011-09-26  (cjwhite)
- Bug fix for checking boundp (myuhe)
- Support special chars (verified titles/notes) (myuhe)
- Added `org-toodledo-inhibit-https' to disable https

2011-09-29  (cjwhite)
- Bug fix: marking a task completed on the server did not update locally

2011-10-16  (cjwhite)
- Bug fix: first time sync of tasks with folders failed with org-mode 6.33x

2011-12-05  (cjwhite)
- Bug fix: folder / id mapping was reversed
- Bug fix: added require for aput / assoc
- Properly clear fields that are not set locally, ensuring they get cleared on server

2012-01-29  (cjwhite) - Version 2.2.0
- Added support for starttime / duetime (merged in partical changes for myuhe).  Note
  that the web site does not seem properly handle timezones.  The time in org-mode
  is properly converted to a unix timestamp (adjusting for timezone) and sent to the
  server, but the displayed time on toodledo.com and in apps (at least iPad app) is
  the time component in GMT, not the timezone set in the account settings.  Waiting
  for a response from toodledo.com on this.

- Added `org-toodledo-flatten-all-tasks' which disables parent/child tasking.
  Set to true if you have more than 2 levels and wish to sync tasks to the server
  as a flat list at least for backup.

- Improved debug logging, use `org-toodledo-toggle-debug' to turn on debug logging

- Updated `org-toodledo-run-tests' to use a test account instead of the users account.

2012-01-30  (cjwhite) - Version 2.3
- Bug fix - major problem found whereby sync / modified times would be off just slightly
  when syncing local changes up to the server.  This would make it look like the local
  task was still modified even after sending it up to the server.  As such, any changes
  to the task on the server would result in duplicate tasks in org-mode on the next
  sync, because that's how changes on both sides are handled.  As part of this fix
  I completely eliminated the per task Modified and Sync properities as these are
  pretty much unneeded because of the detection of changes by the hash code

- Bug fix for startdate / duedate.  These were probably working ok, but may have
  misbehaved if the timezone were more than 12h off GMT, which I think can only happen
  in a very few rare cases.  

- Fixed up the starttime / duetime.  Turns out that toodledo.com treats these times
  like alarms, so a duetime of "7:00am" is 7am, regardless of what timezone you are
  in.  That means you can change your timezone at will and the duetime is still
  7am *local* time.  This is stored as an offset from midnight GMT on the due date.

- Added a version variable / function `org-toodledo-version'.  This is checked on 
  sync and may do some cleanup of things that have changed in various versions.  This
  will make it easier down the road to detect what version someone is running as well.

- Added some delays into the devtest -- seems it may have been syncing back and forth
  too fast (within the same second) such that changes may not be perceived

- Added a lot more debug messages to help with timing / sync problems

2012-02-09  (cjwhite) - Version 2.4
- Added new patch for url-http.  This should address at least some of the spurious
  problems that caused an XML parse error.  If the XML response from the server
  was just around 1200 bytes, url-http was not handling it properly and cutting off
  the last few bytes.  See the Installation instructions below for installing the patches

2012-02-09  (cjwhite) - Version 2.5
- Bug fix - requesting a token was still using https even if `org-toodledo-inhibit-https'
  was set to true.

2012-02-19  (cjwhite) - Version 2.6
- Added `org-toodledo-agenda-mark-task-deleted' function and suggested binding
  to mark tasks deleted from agenda mode.  See Installation for info

- When a task is marked for deleted, move the task to a task titled
  "Deleted Tasks" to get it out of the way

- Handle more than 2-levels of hierarchy by flatting all children.  The hierarchy
  will be preserved.  See SUBTASKS above for more details.

- Realign all tags after sync according to the width of the current window

- Better handling of 'duplicate' tasks -- When a task has been
  modified locally and on the server, the user is prompted to
  resolve the difference by selecting the local copy, the server
  copy, or editing and resolving manually

2012-02-19  (cjwhite) - Version 2.6.1
- Fix for use of http vs https

2012-02-20  (cjwhite) - Version 2.7
- Added support for tags, they are added as org-mode tags.

- Fixed handling of tasks with an embedded '*' - this would
  completely mess things up of the '*' was the first char of
  a line
 
- Added customization `org-toodledo-indent-task-note', set to t
  by default.  This indents the note according to the task level.
  This has a cleaner look and reduces the chance of errors like
  the above with '*' as the first char.

- Better detection of actual note text, eliminating blank lines

- Fixed handling of "Invalid Key" error that indicates the token
  needs to be refreshed

- Fix bug for subtasks if parent is not at top-level

2012-03-13  (cjwhite) - Version 2.8
- Significantly better error handling for adding, editng and deleting
  tasks during synchronization.  If errors occurred, only the offending
  task should be affected, allowing all other tasks to be synced.  The
  specific error message is logged for better debugging.

- Added customization `org-toodledo-folder-support-mode'.  Setting this
  to `heading' will create heading tasks according to the folder name
  and put all tasks in that folder as child tasks of the heading.  
  No support yet for creating folders in org mode, create the folders
  on toodledo.com first and put at least one task in the folder.  This
  feature is somewhat experimental.

- Improved handling of token expiration / invalid key errors

- Added simulation mode for testing without actually hitting the server. 
  Allows simulating error conditions that are potentially hard to 
  reproduce with a real connection to the server.  See org-toodledo-sim.el
  and org-toodledo-test.el where sim calls are used.

2012-07-21  (cjwhite) - Version 2.9
- Added 'length' to the list of fields to ignore if 0 for computing
  a hash

- Removed deprecated/obsolete use of aput/adelete from assoc.el

- Hopefully finally fixed spurious issue where syncing resulted in
  "Failed to find todo entry...".  When processing incoming tasks 
  from the server, process parent tasks *then* child tasks.  It seems
  in some cases the child task may show up first before the parent
  is known.

2013-03-24  (cjwhite) - Version 2.10
- Renamed folder/goal properties to ToodledoFolder and ToodledoGoal

- Added `org-toodledo-reset` function to eliminate any trace of 
  toodledo from an org file (great to use if you want to refresh
  the server from your master org file)

- Fixed a bug where parent/child tasks may not sync properly.  
 
- Improved folder support, works well now supporting creating folders
  and moving tasks around to folders, including subtasks.

- Fixed a bug that was causing issues running tests

