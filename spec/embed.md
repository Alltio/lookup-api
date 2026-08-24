# Mark your pages: the registered-with-Alltio embed

Two lines of code on the page where your work lives — one your readers can see, one AI crawlers can read. Generated from your registry record; nothing to fill in.

The registry answers the question is this on the record? wherever your work travels. The page embed answers it in the one place you control completely: the page where you publish. This page explains what the embed is, where to put it, and — most importantly — what to do when you revise a registered work.

## What the embed is

When a registration is filed, the result screen generates a snippet built from your record. It has two parts:

- The visible notice — one line, placed near the work (a footer under an article, a caption block, a credits line): Registered with Alltio · record 9f86d081… · v2 · registered 2026-07-19 · supersedes record 60303ae2… It links to your public record, so anyone — a reader, an editor, a lawyer on the other side — can verify the claim in one click. The version and supersedes fragments appear only when your record carries them.

- The machine-readable block — a small JSON-LD script (the structured-data format search engines have parsed for over a decade) carrying your record ID, the work's fingerprint, the version, and the link to any superseded record. AI crawlers reading your page find the registration data in the exact format they already consume — no guessing, no lookup required to discover that a record exists.

You don't write either part. Copy the snippet from the registration result screen and paste it into the page's HTML — the same way you'd paste an analytics tag. Done once per page.

## Where to paste it — and what it is not required for

Two things people reasonably assume, both wrong in a good way:

- The embed is not required for your registration to work. Your registration is complete and verifiable the moment the registry countersigns it — the fingerprint lookup finds your work with no embed anywhere on earth. The embed adds speed and visibility on the pages you control; skipping it costs you nothing in validity.

- The machine-readable block does not have to go in the page's `<head>`. JSON-LD is valid anywhere in the HTML — head or body — and crawlers parse it either way. Putting it in the `<head>` is tidy convention; pasting it in the body next to the visible line works identically. If your publishing tool only lets you edit the body (most blog platforms), that is fine.

Placement in practice: put the visible line where a reader would look for a credit — under the article, in a caption block, in the footer of the piece. Put the script block in the `<head>` if you can, or directly beside the visible line if you can't. Both on the same page as the work; that is the only real rule.

## What the embed is not

The embed is an accelerant, not the proof. Your claim is bound by the registry record — the countersigned fingerprint, timestamp, and terms — not by anything on your page. A stripped, copied, or re-hosted file still resolves through the fingerprint lookup with no embed anywhere in sight. What the embed adds is speed and visibility: crawlers discover the registration at the moment they read the page, and humans see that the work has an owner who put it on the record.

One consequence of that: a responsible AI system will verify the embed against the registry rather than trusting the page. That is by design — it means nobody can fabricate an embed claiming your work, because the registry answer, not the page assertion, is what counts.

## Version deliberately — the one rule that keeps your proof intact

Registered works get revised. Essays are updated, reports are corrected, datasets grow. Here is the rule:

The reason is mechanical, not bureaucratic: your registration binds a fingerprint, and any change to the file — one character — produces a different fingerprint. Publish a revised file against an old embed and a verifier sees a mismatch: a page claiming a record whose fingerprint no longer matches the content. You would have broken your own chain of proof.

The correct sequence takes two minutes:

- Register the revised file. On the form, add a version label (“v2”) and paste the record ID from the previous version into “Supersedes.”

- Replace the embed on the page with the newly generated one. The new visible line reads “v2, registered [date], supersedes record #…” — the revision is disclosed, dated, and linked.

The registry does the rest: your new record carries a verifiable back-link to the old one, and the old record is marked as superseded — never edited, never deleted. The result is a dated custody chain through every revision, which is precisely what a dispute, a diligence review, or a licensing negotiation wants to see. A silent edit proves nothing; a version chain proves everything.

## Where the embed fits

## The reference implementation

[maximin.ai](https://maximin.ai) — the publications of Alltio's founding entity — is being built as the reference implementation: every registered page will carry this embed, versioned exactly as described above, from the day it launches. What you will see there is what the snippet produces, maintained by the people who designed it. We eat our own registry first.

Technical readers will find the normative grammar — field by field, including the JSON-LD vocabulary and supersession semantics — in [§5 of the Lookup API specification](/lookup#embed). The plain-language overview of all four discovery layers is at [how discovery works](/discovery).

## Put your knowledge on the record.

Free. Read and fingerprinted in your browser, not stored.
