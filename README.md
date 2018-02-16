[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.tdomzal/junit-docker-rule/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.tdomzal/junit-docker-rule/)
[![Build Status](https://travis-ci.org/tdomzal/junit-docker-rule.svg?branch=master)](https://travis-ci.org/tdomzal/junit-docker-rule)
[![codecov.io](https://codecov.io/github/tdomzal/junit-docker-rule/coverage.svg?branch=master)](https://codecov.io/github/tdomzal/junit-docker-rule?branch=master)

# junit-docker-rule #

## Changelog ##
see [changelog](doc/changelog.md)

## What is it ? ##

Simple [JUnit Rule](https://github.com/junit-team/junit/wiki/Rules) starting [docker](https://www.docker.com/) container right for your test case:

    package org.example;

    import static org.hamcrest.CoreMatchers.containsString;
    import static org.junit.Assert.assertThat;

    import java.io.IOException;
    import org.apache.http.client.fluent.Request;
    import org.junit.Rule;
    import org.junit.Test;
    import pl.domzal.junit.docker.rule.DockerRule;

    public class HomepageExampleTest {

        @Rule
        public DockerRule container = DockerRule.builder() //
                .imageName("nginx") //
                .build();

        @Test
        public void shouldExposePorts() throws InterruptedException, IOException {

            // url container homepage will be exposed under
            String homepage = "http://"+container.getDockerHost()+":"+container.getExposedContainerPort("80")+"/";

            // use fluent apache http client to retrieve homepage content
            String pageContent = Request.Get(homepage).connectTimeout(1000).socketTimeout(1000).execute().returnContent().asString();

            // make sure this is indeed nginx welcome page
            assertThat(pageContent, containsString("Welcome to nginx!"));
        }

    }

Container is started just before your test case and destroyed after.

It was created as side product and I'll be more than happy if you'll find it useful !

## What docker options it currently supports ? ##

You can:

- use it as JUnit @Rule or @ClassRule
- specify image name/tag
- specify container name (equivalent of command line `--name`)
- define links to other containers (`--link`)
- pass environment variables (`--env` or `-e`)
- publish all exposed port to dynamically allocated host ports (`--publish-all` or `-P`)
- publish specified container ports to specified host ports (`-p` - tcp or udp, no port
  ranges support yet)
- mount host directory as a data volume (`--volume` or `-v` - also works for workstation
  dirs to boot2docker container with restriction that dir must be under user homedir)
- specify extra /etc/hosts entries (`--add-host`)
- access container stderr and stdout (forwarded to java System.err and System.out by
  default)
- wait for message or sequence of messages in container output at container start

See usage [examples](src/test/java/pl/domzal/junit/docker/rule/examples/) in test cases.

Also - just type `DockerRule.builder().` .. and try code assist in your favorite IDE (Alt-Enter in IDEA, Ctrl-Space in Eclipse) to see all possible options.

## What do I need to use it ? ##

### 1. Install Docker (of course) ###

Docker should be installed and configured - which in general means you must have docker variables (DOCKER\_HOST, DOCKER\_MACHINE\_NAME, ...) available in runtime. Preferred way to set them is via [docker-machine](https://docs.docker.com/machine/) command.

### 2. Declare dependency in pom.xml ###

    ...
    <dependency>
        <groupId>com.github.tdomzal</groupId>
        <artifactId>junit-docker-rule</artifactId>
        <version>0.4</version>
        <scope>test</scope>
    </dependency>
    ...

### 3. Use in test case ###

    import pl.domzal.junit.docker.rule.DockerRule;

    public class MyTestCase {
        ...
        @Rule
        public DockerRule testee = DockerRule.builder()
            .imageName("nginx")
            // ... other build options (try code assist from your IDE to explore available options)
            .build();
        ...
        // your test cases
    }

## What else should I know for now ? ##

- It uses java [docker client from Spotify](https://github.com/spotify/docker-client)
- Build and tested with docker 1.9
- This is work in progress (but all features are verified by tests)

## How to use latest (SNAPSHOT) version ##

Change dependency to:

    ...
    <dependency>
        <groupId>com.github.tdomzal</groupId>
        <artifactId>junit-docker-rule</artifactId>
        <version>0.5-SNAPSHOT</version>
        <scope>test</scope>
    </dependency>
    ...

Add OSS Sonatype snapshot repository to you *pom.xml*:

    <project>
        ...
        <repositories>
            ...
            <repository>
                <id>ossrh</id>
                <releases><enabled>false</enabled></releases>
                <snapshots><enabled>true</enabled></snapshots>
                <url>https://oss.sonatype.org/content/repositories/snapshots</url>
            </repository>
            ...
        </repositories>
        ...
    </project>

## How to build ? ##

Assuming you have apache maven installed it's just:

    git clone https://github.com/tdomzal/junit-docker-rule.git
    cd junit-docker-rule
    mvn install -DskipTests

or, if you want to build with tests - substitute last command with:

    mvn install

Of course test cases will run only if you have working docker environment.
