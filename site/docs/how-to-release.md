<!--
 - Licensed to the Apache Software Foundation (ASF) under one or more
 - contributor license agreements.  See the NOTICE file distributed with
 - this work for additional information regarding copyright ownership.
 - The ASF licenses this file to You under the Apache License, Version 2.0
 - (the "License"); you may not use this file except in compliance with
 - the License.  You may obtain a copy of the License at
 -
 -   http://www.apache.org/licenses/LICENSE-2.0
 -
 - Unless required by applicable law or agreed to in writing, software
 - distributed under the License is distributed on an "AS IS" BASIS,
 - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 - See the License for the specific language governing permissions and
 - limitations under the License.
 -->

## Setup

To create a release candidate, you will need:

* Apache LDAP credentals for Nexus and SVN
* A [GPG key for signing](https://www.apache.org/dev/release-signing#generate), published in [KEYS](https://dist.apache.org/repos/dist/dev/iceberg/KEYS)

If you have not published your GPG key yet, you must publish it before sending the vote email by doing:

```shell
svn co https://dist.apache.org/repos/dist/dev/iceberg icebergsvn
cd icebergsvn
echo ""  >> KEYS # append a newline
gpg --list-sigs <YOUR KEY ID HERE> >> KEYS # append signatures
gpg --armor --export <YOUR KEY ID HERE> >> KEYS # append public key block
svn commit -m "add key for <YOUR NAME HERE>"
```

### Nexus access

Nexus credentials are configured in your personal `~/.gradle/gradle.properties` file using `mavenUser` and `mavenPassword`:

```
mavenUser=yourApacheID
mavenPassword=SomePassword
```

### PGP signing

The release scripts use the command-line `gpg` utility so that signing can use the gpg-agent and does not require writing your private key's passphrase to a configuration file.

To configure gradle to sign convenience binary artifacts, add the following settings to `~/.gradle/gradle.properties`:

```
signing.gnupg.keyName=Your Name (CODE SIGNING KEY)
```

To use `gpg` instead of `gpg2`, also set `signing.gnupg.executable=gpg`

For more information, see the Gradle [signing documentation](https://docs.gradle.org/current/userguide/signing_plugin.html#sec:signatory_credentials).

## Creating a release candidate

### Build the source release

To create the source release artifacts, run the `source-release.sh` script with the release version and release candidate number:

```bash
dev/source-release.sh -v 0.13.0 -r 0 -k <YOUR KEY ID HERE>
```

Example console output:

```text
Preparing source for apache-iceberg-0.13.0-rc1
Adding version.txt and tagging release...
[master ca8bb7d0] Add version.txt for release 0.13.0
 1 file changed, 1 insertion(+)
 create mode 100644 version.txt
Pushing apache-iceberg-0.13.0-rc1 to origin...
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 12 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 433 bytes | 433.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/apache/iceberg.git
 * [new tag]           apache-iceberg-0.13.0-rc1 -> apache-iceberg-0.13.0-rc1
Creating tarball  using commit ca8bb7d0821f35bbcfa79a39841be8fb630ac3e5
Signing the tarball...
Checking out Iceberg RC subversion repo...
Checked out revision 52260.
Adding tarball to the Iceberg distribution Subversion repo...
A         tmp/apache-iceberg-0.13.0-rc1
A         tmp/apache-iceberg-0.13.0-rc1/apache-iceberg-0.13.0.tar.gz.asc
A  (bin)  tmp/apache-iceberg-0.13.0-rc1/apache-iceberg-0.13.0.tar.gz
A         tmp/apache-iceberg-0.13.0-rc1/apache-iceberg-0.13.0.tar.gz.sha512
Adding         tmp/apache-iceberg-0.13.0-rc1
Adding  (bin)  tmp/apache-iceberg-0.13.0-rc1/apache-iceberg-0.13.0.tar.gz
Adding         tmp/apache-iceberg-0.13.0-rc1/apache-iceberg-0.13.0.tar.gz.asc
Adding         tmp/apache-iceberg-0.13.0-rc1/apache-iceberg-0.13.0.tar.gz.sha512
Transmitting file data ...done
Committing transaction...
Committed revision 52261.
Creating release-announcement-email.txt...
Success! The release candidate is available here:
  https://dist.apache.org/repos/dist/dev/iceberg/apache-iceberg-0.13.0-rc1

Commit SHA1: ca8bb7d0821f35bbcfa79a39841be8fb630ac3e5

We have generated a release announcement email for you here:
/Users/jackye/iceberg/release-announcement-email.txt

Please note that you must update the Nexus repository URL
contained in the mail before sending it out.
```

The source release script will create a candidate tag based on the HEAD revision in git and will prepare the release tarball, signature, and checksum files. It will also upload the source artifacts to SVN.

Note the commit SHA1 and candidate location because those will be added to the vote thread.

Once the source release is ready, use it to stage convenience binary artifacts in Nexus.

### Build and stage convenience binaries

Convenience binaries are created using the source release tarball from in the last step.

Untar the source release and go into the release directory:

```bash
tar xzf apache-iceberg-0.13.0.tar.gz
cd apache-iceberg-0.13.0
```

To build and publish the convenience binaries, run the `dev/stage-binaries.sh` script. This will push to a release staging repository.

```
dev/stage-binaries.sh
```

Next, you need to close the staging repository:

1. Go to [Nexus](https://repository.apache.org/) and log in
2. In the menu on the left, choose "Staging Repositories"
3. Select the Iceberg repository
4. At the top, select "Close" and follow the instructions
    * In the comment field use "Apache Iceberg &lt;version&gt; RC&lt;num&gt;"

!!! Note
   If multiple staging repositories are created after running the script, set `org.gradle.parallel=false` in `gradle.properties`

### Start a VOTE thread

The last step for a candidate is to create a VOTE thread on the dev mailing list.
The email template is already generated in `release-announcement-email.txt` with some details filled.

Example title subject:

```text
[VOTE] Release Apache Iceberg <VERSION> RC<NUM>
```

Example content:

```text
Hi everyone,

I propose the following RC to be released as official Apache Iceberg <VERSION> release.

The commit id is <SHA1>
* This corresponds to the tag: apache-iceberg-<VERSION>-rc<NUM>
* https://github.com/apache/iceberg/commits/apache-iceberg-<VERSION>-rc<NUM>
* https://github.com/apache/iceberg/tree/<SHA1>

The release tarball, signature, and checksums are here:
* https://dist.apache.org/repos/dist/dev/iceberg/apache-iceberg-<VERSION>-rc<NUM>/

You can find the KEYS file here:
* https://dist.apache.org/repos/dist/dev/iceberg/KEYS

Convenience binary artifacts are staged in Nexus. The Maven repository URL is:
* https://repository.apache.org/content/repositories/orgapacheiceberg-<ID>/

This release includes important changes that I should have summarized here, but I'm lazy.

Please download, verify, and test.

Please vote in the next 72 hours.

[ ] +1 Release this as Apache Iceberg <VERSION>
[ ] +0
[ ] -1 Do not release this because...
```

When a candidate is passed or rejected, reply with the voting result:

```text
Subject: [RESULT][VOTE] Release Apache Iceberg <VERSION> RC<NUM>
```

```text
Thanks everyone who participated in the vote for Release Apache Iceberg <VERSION> RC<NUM>.

The vote result is:

+1: 3 (binding), 5 (non-binding)
+0: 0 (binding), 0 (non-binding)
-1: 0 (binding), 0 (non-binding)

Therefore, the release candidate is passed/rejected.
```


### Finishing the release

After the release vote has passed, you need to release the last candidate's artifacts.

First, copy the source release directory to releases:

```bash
mkdir iceberg
cd iceberg
svn co https://dist.apache.org/repos/dist/dev/iceberg candidates
svn co https://dist.apache.org/repos/dist/release/iceberg releases
cp -r candidates/apache-iceberg-<VERSION>-rcN/ releases/apache-iceberg-<VERSION>
cd releases
svn add apache-iceberg-<VERSION>
svn ci -m 'Iceberg: Add release <VERSION>'
```

Next, add a release tag to the git repository based on the passing candidate tag:

```bash
git tag -am 'Release Apache Iceberg <VERSION>' apache-iceberg-<VERSION> apache-iceberg-<VERSION>-rcN
```

Then release the candidate repository in [Nexus](https://repository.apache.org/#stagingRepositories).

To announce the release, wait until Maven central has mirrored the Apache binaries, then update the Iceberg site and send an announcement email:

```text
[ANNOUNCE] Apache Iceberg release <VERSION>
```
```text
I'm please to announce the release of Apache Iceberg <VERSION>!

Apache Iceberg is an open table format for huge analytic datasets. Iceberg
delivers high query performance for tables with tens of petabytes of data,
along with atomic commits, concurrent writes, and SQL-compatible table
evolution.

This release can be downloaded from: https://www.apache.org/dyn/closer.cgi/iceberg/<TARBALL NAME WITHOUT .tar.gz>/<TARBALL NAME>

Java artifacts are available from Maven Central.

Thanks to everyone for contributing!
```
