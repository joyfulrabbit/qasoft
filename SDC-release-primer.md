# Docs
[sdc-release](https://mo.joyent.com/docs/engdoc/master/sdcrelease/index.html) <br>
[staging](https://mo.joyent.com/docs/lab/master/staging.html)<br>
[update](https://github.com/joyent/sdcadm/blob/master/docs/update.md)<br>
# Mattermost Channels to add:
~nightly <br>
~triton-dev <br>
~os <br>

# Subscribe to mailing lists
In order to send out the release emails you'll need to join mailing lists (if you aren't already a member).<br>

Use this URL to add yourself:<br>
sdc-discuss@lists.smartdatacenter.org now at https://smartdatacenter.topicbox.com/groups/sdc-discuss <br>
smartos-discuss@lists.smartos.org now at https://smartos.topicbox.com/groups/smartos-discuss <br>





# Useful Variables

_ephemeral (per release )_:
```
export RELEASE=release-YYYYMMDD
export LASTRELEASE=release-YYYYMMDD
```
_permanent (.bash_profile)_:
```
export MOLYBDENUM_CREDENTIALS="get USERNAME:PASSWORD from qa teammate"
export JENKINS_AUTH="<user>:<api token>"
export UPDATES_IMGADM_USER="<yourimgadmusername>"
export UPDATES_IMGADM_IDENTITY="$(ssh-keygen -l -f $HOME/.ssh/id_rsa_imgadm.pub | awk '{print $2}')"
```
# Software
Before you begin, make sure you have NPM installed on some reasonable version that isn't super old. I strongly recommend a clean install using nvm, as it builds all your binaries in a version specific folder. It allows you jump between different npm builds without breaking your permissions or throwing "package compiled with wrong node version" errors.
Make sure npm manta and json packages are setup.
Make sure you have /Joyent_Dev and /Joyent_SW accessible via manta.

## _JIRASH (install it):_
```
$ npm install -g jirash
$ cat ~/.jirash.json
{
    "jira_url": "https://jira.joyent.us",
    "jira_username": "your-username",
    "jira_password": "your-password"
}
```
## _mountain gorilla (install it):_
```
git clone git@github.com:joyent/mountain-gorilla.git
cd mountain-gorilla
export MOLYBDENUM_CREDENTIALS="get USERNAME:PASSWORD from Trent"
./tools/check-repos-for-release -a ${RELEASE:8}
export MOLYBDENUM_CREDENTIALS="molybdenum:<askme>"
```
## _JENKINS API:_
You'll need to provide the launch-build program with your Jenkins API token. You can find this by clicking on your username in the Jenkins web interface, then 'Configure', and then 'Show API Token'. You can then put it in your environment as:
export JENKINS_AUTH="<user>:<api token>"
where the user is the username that you use to log into Jenkins -not- the name you have given the token.

## _ENGADM:_
```
git clone git@git.joyent.com:engadm.git
cd engadm
make
./bin/engadm release-sdc
```
## _Updates.joyent.com:_

This key MUST NOT have a passphrase. I had to make a new one!

1. Add an sshkey to the /Joyent_SW manta path with the following format:
```
[us-east.manta.joyent.com /Joyent_SW/stor/updates.joyent.com/authkeys]$ ls
...
trentm.keys
```
(just paste a public key WITH NO PASSWORD into a file <yourname>.keys)
The name of the file will be used as your IMGADM_USER in the next step.
Note, it takes some time for this file to be scraped.

2.Setup to use updates-imgadm (the CLI for updates.joyent.com). You can run this for your Mac or whereever:
```
git clone git@github.com:joyent/sdc-imgapi-cli.git
cd sdc-imgapi-cli
make
export UPDATES_IMGADM_USER="your updates.jo username"
export UPDATES_IMGADM_IDENTITY="your pubkey signature a la MANTA_KEY_ID"
export PATH=$PATH:`pwd`/bin
```
__If your SSH-KEY has a password it throws errors like so:__
```
ryankitchen$ ./updates-imgadm ping assert.js:351 throw err; ^ AssertionError [ERR_ASSERTION]: options.public must be a sshpk.Key instance at AgentKeyPair.SDCKeyPair (/Users/ryankitchen/github/sdc-imgapi-cli/node_modules/smartdc-auth/lib/keypair.js:50:14) at new AgentKeyPair (/Users/ryankitchen/github/sdc-imgapi-cli/node_modules/smartdc-auth/lib/kr-agent.js:93:10) at /Users/ryankitchen/github/sdc-imgapi-cli/node_modules/smartdc-auth/lib/kr-agent.js:81:14 at Request.r_cb (/Users/ryankitchen/github/sdc-imgapi-cli/node_modules/sshpk-agent/lib/client.js:67:3) at Request.state_done (/Users/ryankitchen/github/sdc-imgapi-cli/node_modules/sshpk-agent/lib/client-fsm.js:224:8) at Request.FSM._gotoState (/Users/ryankitchen/github/sdc-imgapi-cli/node_modules/mooremachine/lib/fsm.js:336:4) at FSMStateHandle.gotoState (/Users/ryankitchen/github/sdc-imgapi-cli/node_modules/mooremachine/lib/fsm.js:77:23) at Client.<anonymous> (/Users/ryankitchen/github/sdc-imgapi-cli/node_modules/sshpk-agent/lib/client-fsm.js:190:5) at Client.emit (events.js:197:13) at AgentDecodeStream.<anonymous> (/Users/ryankitchen/github/sdc-imgapi-cli/node_modules/sshpk-agent/lib/client-fsm.js:369:9)
```
