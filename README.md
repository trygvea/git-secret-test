
## Remove test container
Also take a look at  'Reset repo' below

    docker stop gitsecret
    docker container rm gitsecret
    docker image rm gitsecret-test


## Install test container
    docker build -t gitsecret-test .
    docker run -it --name gitsecret -d gitsecret-test:latest
    docker exec -it gitsecret bash

    # Install git-secret on docker host
    wget https://bintray.com/sobolevn/rpm/rpm -O bintray-sobolevn-rpm.repo
    mv bintray-sobolevn-rpm.repo /etc/yum.repos.d/
    yum install git-secret


## Clone repo and define gpg user. From docker host:
    git clone https://github.com/trygvea/git-secret-test.git
    cd git-secret-test
    
    # define gpg user on docker host
    gpg --batch --gen-key docker-test-images/gpg-create-user.txt
    gpg --export -a gitsecret-test@foo.com > PUBLIC-KEY.txt
    

## Test
    # Check that we have no access on docker host:
    cat secret.txt	# should not exist
    git secret reveal   # should giva an error

    # give test user access to secret. From local host run:
    gpg --import PUBLIC-KEY.txt		          # copy generated key from docker host
    git secret tell gitsecret-test@foo.com    # give access. You may be asked for passphrase here
    git secret hide -d  # NOTE: This steps seems unneccessary, but are essential 
    git secret reveal   # NOTE: This steps seems unneccessary, but are (probably) essential
    git add .
    git commit -m "Revealing secret to test user. NOTE: remove this commit for repeated testing and `git push -f`"
    git push

    # Test if we can reveal secret now, on docker host:
    git pull
    git secret tell gitsecret-test@foo.com    # must also give ourselves access here, weird
    git secret reveal
    cat secret.txt    # Now we should see the secret text!


## Reset repo
For repeated testing, the commit revealing the secret should be removed.

    git log		# Make sure the "Reveal secret..." commit is the last commit
    git reset --hard HEAD~
    git push -f


