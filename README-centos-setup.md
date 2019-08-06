# Inn i Confluence på linuxserver148:

# Installering av git-secret og gpg på ny jenkins server
Git secret brukes for å skjule hemmeligheter for ikke-autoriserte brukere, samtidig 
som disse hemmelighetene ligger kryptert i git repoet.

Beskrivelsen under er hva som er gjort på jenkins server. 
Det meste av dette er tatt fra https://git-secret.io/ og https://www.gnupg.org/documentation/.


## Installer git-secret
Dårlig støtte for gpg på CentOS. (Andre maskiner gjør bare brew/apt-get install git-secret)
 
    wget https://bintray.com/sobolevn/rpm/rpm -O bintray-sobolevn-rpm.repo
    sudo mv bintray-sobolevn-rpm.repo /etc/yum.repos.d/
    sudo yum install git-secret


## Definer gpg-bruker jenkins-trafikksystemer@vy.no
Denne trenger vi for at bygg skal kunne hente frem krypterte filer med  `git secret reveal`.

CentOS har bare støtte for gpg versjon 2.0, og denne fremgangsmåten er spesialsydd
for denne. Se forøvrig https://www.gnupg.org/documentation/manuals/gnupg/Unattended-GPG-key-generation.html.

Merk spesiellt at det ikke er passphrase for denne nøkkelen.  

    sudo -i
    cat >foo <<EOF
     %echo Generating a basic OpenPGP key
     %no-ask-passphrase
     Key-Type: DSA
     Key-Length: 4096
     Subkey-Type: default
     Name-Real: Jenkins Trafikksystemer
     Name-Comment: Enable revealing secrets on build server
     Name-Email: jenkins.trafikksystemer@vy.no
     Expire-Date: 0
     # Do a commit here, so that we can later print "done" :-)
     %commit
     %echo done
    EOF
    su jenkins -s /bin/bash -c 'gpg --batch --gen-key /root/foo'
    rm foo 


## Hent ut public key:
Denne nøkkelen brukes av prosjekter som vil gi jenkins tilgang til secrets.
 
    su jenkins -s /bin/bash -c 'gpg --export -a jenkins.trafikksystemer@vy.no'

som i skrivende stund gir oss følgende nøkkel:

    -----BEGIN PGP PUBLIC KEY BLOCK-----
    Version: GnuPG v2.0.14 (GNU/Linux)
    
    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
    cN115ue/UsndsCVeUnB6ugCRS64=
    =cjbx
    -----END PGP PUBLIC KEY BLOCK-----

Denne kan feks lagres som jenkins-public-key.txt og importeres med

    gpg --import jenkins-public-key.txt


## Gi jenkins bygg tilgang til secrets
Hvert bygg som trenger tilgang på secrets må:

Fortelle prosjektet at jenkins.trafikksystmer@vy.no skal kunne se secrets. 
En bruker som allerede har rettigher må gjøre følgende (på lokal maskin):

    gpg --import jenkins-public-key.txt
    git secret tell jenkins.trafikksystmer@vy.no
    git secret hide -d
    git secret reveal
    git add .
    git commit -m "Give Jenkins server access to secrets"
    git push
    
samt lage et bygge-step på jenkins med 'Execute shell' med følgende kommandoer:
 
    git secret tell jenkins.trafikksystmer@vy.no
    git secret reveal
    

## Ta backup av private key:
Backup av private key kan være smart, men denne må oppbevares på dertil egnet måte.
Den kan feks brukes om jenkins server må reinstalleres fra scratch.

    (sudo -i)
    su jenkins -s /bin/bash -c 'gpg --export-secret-keys -a jenkins.trafikksystemer@vy.no'


## Prosjektoppsett
Nye prosjekter som skal ta i bruk git-secret bør følge fremgangsmåten på https://git-secret.io/ 


