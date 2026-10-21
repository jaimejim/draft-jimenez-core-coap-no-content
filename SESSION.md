# Session - 2026-10-21

## What happened
Wrote the -00 of draft-jimenez-core-coap-no-content from Mike Bishop's AD review
of draft-ietf-core-coap-pubsub-21 (item 6, HALF CREATED state) and
core-wg/coap-pubsub issue 4 (Koster's 2.06 proposal). The draft defines
2.06 (No Content): resource exists, no current representation. GET and
FETCH only; no payload, Content-Format or ETag; cacheable with Max-Age;
updates RFC 7641 so a 2.06 with Observe confirms a registration and a 2.06
notification keeps the observer and invalidates the old value; maps to and
from HTTP 204 (adds a row to RFC 8075 Table 2). Examples follow
draft-jimenez-core-coap-diag (4-tilde `coap` fences). Appendix lists
rejected alternatives (2.05 empty, 4.04, 5.03, placeholder data, 2.04).

Then found Carsten's parallel skeleton, created the same morning:
https://github.com/cabo/no-content (draft-bormann-core-no-content). Same
idea, mostly TODOs, but it has 3 extra alternatives (media type's own
null value, a "null media type" vs Accept, RFC 8710 multipart-core) and the
full RFC 7252 §12.1.2 registration checklist. Recommendation given: merge
into one draft rather than two -00s; user has not decided which repo wins.

ETag discussion: HTTP 204 may carry ETag (RFC 9110 §15.3.5), our draft
bans it. Consequences to state: no revalidation of a 2.06, no 2.03 for
the empty state. Open gap in both drafts: If-None-Match semantics on an
empty resource (RFC 7252 §5.10.8.2 "does not exist" vs RFC 9110 §13.1.2
"no current representation"). Proposed resolving it the HTTP way.

## Changes made
Repo: jaimejim/draft-jimenez-core-coap-no-content (private), branch main.
- 2981cfc  initial -00 (.md, README.md, .gitignore)
- c85b0af  empty commit (git commit -a quirk, see below)
- 08cc3db  rendered .txt committed, .gitignore exception for it
- c16270b  Carsten Bormann added as co-author (block copied from coap-diag)
- 600a270  empty commit (same quirk)
- b648aa8  author name -> Jaime Jiménez
- c56014c  pubsub and coap-diag references written out by hand so they
           read "Jiménez"; they override datatracker bibxml and must be
           bumped manually (pubsub pinned at -21)
Build: `kdrfc -3 -h draft-jimenez-core-coap-no-content.md`, clean apart
from the 2 expected "explicit settings override" warnings.
No change to coap-pubsub or coap-diag.

## Open threads
- Merge with cabo/no-content: PR our text into his repo under his docname,
  or he joins ours. Needs Jaime + Carsten.
- Carsten has not been asked whether he agrees to be listed as co-author.
- Repo is private; make public or grant access before pointing core@ to it.
- `area: Applications` is outdated; Carsten uses "Web and Internet
  Transport". Fix before submission.
- Missing from IANA/spec text vs RFC 7252 §12.1.2 checklist: validatable
  (no, stated only implicitly), and whether a 2.06 marks stored responses
  for the URI as not fresh (should say: replaces them like a 2.05).
- Add Carsten's 3 alternatives to the appendix.
- If-None-Match on an empty resource: decide and write one sentence.
- "Updates: 7641" only; RFC 8075 Table 2 also gains a row.
- Two empty commits on main (c85b0af, 600a270); removing them needs a
  force-push, user has not asked.
- coap-pubsub and coap-diag sources still spell "Jimenez"; user not yet
  asked whether to change them.
- Mike's item 6 in coap-pubsub/reply-to-mike-ad-review.md is still TODO;
  the reply should point at whichever no-content draft survives.
- `git commit -a` made empty commits twice with the file open in Kiro;
  always `git add <file>` explicitly in this repo.

## Next steps
1. Ask Carsten: merge direction, co-authorship, area string.
2. Add the §12.1.2 checklist answers, If-None-Match sentence, and the 3
   extra alternatives.
3. Fill Jaime's reply to Mike item 6 with a pointer to the draft.
4. Decide on history cleanup and repo visibility, then announce on core@.
