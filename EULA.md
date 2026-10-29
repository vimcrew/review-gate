# VimCrew Review Gate — End User License Agreement (container image)

> **DRAFT — requires owner/legal review.** This text is a working draft prepared together with the
> code. It is not in force and must not be published or relied upon until the owner and a lawyer
> have reviewed it. Placeholders in square brackets must be filled in.

Version: draft-0 · Date: [EFFECTIVE DATE]

This End User License Agreement ("EULA") is between [LICENSOR LEGAL NAME], [JURISDICTION AND
REGISTERED ADDRESS] ("VimCrew", "we") and the person or entity that downloads, pulls or runs the
software ("you").

## 1. What this EULA covers

1.1. "Software" means the container image published as `ghcr.io/vimcrew/review-gate` (any tag or
digest), the VimCrew Review Gate program inside it and its documentation, excluding the
Open-Source Components (section 6).

1.2. The GitHub Action that pulls and runs the image (the files of the `vimcrew/review-gate`
repository: `action.yml`, `README.md`, examples) is licensed separately under the Apache License
2.0 and is not covered by this EULA.

1.3. By pulling or running the Software, including through the GitHub Action, you accept this
EULA. If you do not accept it, do not pull or run the Software.

## 2. License grant

2.1. Subject to this EULA, VimCrew grants you a non-exclusive, non-transferable, non-sublicensable,
revocable, royalty-free license to run the Software, unmodified, to review changes (pull requests)
in source code repositories that you own or are authorized to work on, including in continuous
integration jobs of your organization.

2.2. [OPEN QUESTION FOR THE OWNER: free for all uses, or free with limits (e.g. number of private
repositories / commercial use) and a paid tier? Section 2.1 currently assumes free use.]

## 3. Restrictions

You must not, and must not allow others to:

(a) copy, redistribute, sell, rent, lease or sublicense the Software, or publish it in another
registry, except for transient copies (caches, mirrors) made solely to run it under section 2;

(b) offer the Software, or its functionality, to third parties as a hosted or managed service;

(c) decompile, disassemble or reverse engineer the Software, or extract its program code, except
to the extent such restriction is prohibited by applicable law;

(d) modify the Software, or create derivative works of it;

(e) remove or alter any proprietary, license or attribution notices in the Software;

(f) use the Software in breach of applicable law or of the terms of the model providers you
connect it to.

## 4. Your API keys, model providers and your data

4.1. The Software works only with API keys that you supply ("bring your own keys"). Requests to
model providers (for example Anthropic, OpenAI, Z.ai, Moonshot, MiniMax, DeepSeek, Alibaba Cloud)
are sent from your environment directly to the providers you configure, under your accounts and
your agreements with those providers. VimCrew is not a party to those agreements and is not
responsible for the providers' services, availability, pricing or handling of your data.

4.2. The Software sends the content of the reviewed change (diff, title, description) only to the
model providers you configure, and posts results only to the code hosting platform the job runs
against. It does not send your source code or review texts to VimCrew.

4.3. Usage statistics. The Software sends no telemetry to VimCrew unless you explicitly enable it.
Any statistics feature will be opt-in, will exclude source code and review texts, and will be
described in the documentation before it is offered.

4.4. You are responsible for keeping your API keys and tokens secret and for choosing providers
whose data terms are acceptable for your code.

## 5. Ownership

The Software is licensed, not sold. VimCrew and its licensors retain all rights, title and
interest in the Software. No rights are granted except those expressly stated in this EULA.

## 6. Open-Source Components

The image contains third-party open-source components (including the Eclipse Temurin Java runtime,
the GitHub CLI `gh`, the base operating system packages, and Java/Kotlin libraries). Each is
licensed under its own license ("Open-Source Components"), and nothing in this EULA limits your
rights under those licenses. [TODO before release: ship a third-party notices file inside the image
and reference its path here.]

## 7. Updates

VimCrew may publish new versions of the Software. Each version is identified by its digest; a
version you pinned does not change. This EULA applies to each version unless a different EULA is
published with it.

## 8. Disclaimer of warranties

THE SOFTWARE IS PROVIDED "AS IS" AND "AS AVAILABLE", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NON-INFRINGEMENT. AUTOMATED REVIEW CAN MISS DEFECTS AND VULNERABILITIES AND CAN REPORT FALSE
FINDINGS; IT DOES NOT REPLACE YOUR OWN REVIEW AND TESTING.

## 9. Limitation of liability

TO THE MAXIMUM EXTENT PERMITTED BY LAW, VIMCREW WILL NOT BE LIABLE FOR ANY INDIRECT, INCIDENTAL,
SPECIAL, CONSEQUENTIAL OR PUNITIVE DAMAGES, OR FOR LOSS OF PROFITS, DATA OR GOODWILL, OR FOR
CHARGES OF MODEL PROVIDERS, ARISING FROM THE USE OF THE SOFTWARE. VIMCREW'S TOTAL LIABILITY UNDER
THIS EULA IS LIMITED TO [AMOUNT, e.g. USD 100].

## 10. Termination

This EULA is effective until terminated. It terminates automatically if you breach it. On
termination you must stop running the Software and delete your copies of it. Sections 3, 5, 8, 9
and 11 survive termination.

## 11. Governing law

This EULA is governed by the laws of [GOVERNING LAW], and disputes are subject to the exclusive
jurisdiction of the courts of [VENUE], unless mandatory law provides otherwise.

## 12. Contact

[CONTACT EMAIL / ADDRESS]
