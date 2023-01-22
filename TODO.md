# ToDo

## Manage the "variables" better
I probably should write out a "vars file", created at the time this project was started.  The reason being:  If... I start the project on 1-20, then MY_DATE=2023-01-20.  And if I don't finish on 1-20 and reconvene on 1-23 and resource the variables.txt file, it will update MY_DATE to 2023-01-23 - yet all the previously created services, etc... were created using 2023-01-21.
*possibly* add the vars to the ~/.bashrc - OR... create a file ~/vars.txt and then source it from .bashrc
echo "Sourcing Project Vars: ~/vars.txt" | tee -a ~/.bashrc
echo ". ~/vars.txt" | tee -a ~/.bashrc

## Install latest EKS version
Currently my code is hardcoded to a version.  Probably should make it default to "current".
