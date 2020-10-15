# Project Name: docker-cgi

## BRIEF DESCRIPTION:
  * an Apache module that allows a user to run one or more CGI programs from within a container.

## CURRENT STATUS (MVP 1):
  * An "Apache Handler" is provided to exec the CGI program from within a container.
  * The handler can, as necessary, build, create, and start the appropriate docker images and containers.
  * The handler, which can also be called from the CLI, provides directives to manage the associated images and containers
  * A set of examples are provided via the [docker-cgi.examples project](https://github.com/csuntechlab/docker-cgi.examples).
  * A website that contains the docker-cgi.examples: https://www.sandbox.csun.edu/~steve/docker-cgi.

## PURPOSE:
  * To provide a developer with total control over their programming statck, independent of the web-server environment
  * To create isolation between the user's code and the main OS environment
  * To eliminate the need to deploy a user's container on an independent web-server

## SERVER INSTALLATION:
  1. Install docker on your server
  1. Copy docker-cgi.cgi into a well known web-assessible directory, e.g., /usr/lib/cgi-bin/
  1. Set appropriate permissions for the docker-cgi.cgi script to be executed
  1. Grant password-less sudo permissions to the docker commands for the web server
  	* www-data ALL=(ALL) NOPASSWD: /usr/bin/docker

## USER USAGE NOTES:
  * Develop a git repo that contains your application that can be deployed via a dockerfile
  * Create an appropriate .docker.cgi configuration file
  * Add the following directive to the web folder's .htaccess file
    ```
      AddHandler dockerfile .docker-cgi
      Action dockerfile "/cgi-bin/docker-cgi.cgi"
    ```
  * A Client can access the CGI program via the URL: https://www.domain.com/~/directory/program.docker-cgi
  
	
## USAGE:
```$ docker-cgi help
Usage: docker-cgi                    # When called via the Apache Handler
Usage: docker-cgi DIRECTIVE FILE
Usage: docker-cgi help

  DIRECTIVE is an action to be taken. A list of available DIRECTIVEs are provided below
  FILE is the name of a configuration file that used to invoke the Apache Handler
      This file has the extension of .docker-cgi

      The following environment variables are expected to be defined:
         PATH_INFO:		          the URI path of the .docker-cgi file
         PATH_TRANSLATED:    the file system path of the .docker-cgi file
         REDIRECT_HANDLER:   the name of the handler: i.e., docker-cgi

Available DIRECTIVEs include:

	 help:	  to provide this usage message
	 build:	  to build the a local image from the CONTEXT
	 rmi:	  to remove the image associated with the CONTEXT
	 create:  to create the container from the associated image
	 start:	  to start the associated container
	 exec:	  to execute a CGI program defined by the ENTRY variable
	 stop:	  to start the associated container
	 rm:	  to remove the associated container
	 headers: to emit the HTTP response headers defined within FILE
	 envs:	  to emit the environment variables that will be passed to to ENTRY program
	 env_args: to emit the environment variables as used as arguments to docker exec

For testing purposes, this script can be invoked as if it were being called by the Apache server as follows:

	 $ PATH_TRANSLATED="$FILE" docker-cgi 

Caveat/Bugs:
	 - When invoked as a CGI Handler from the command line:
	   the PATH_INFO will not be set correctly.
	 - If PATH_INFO or REDIRECT_HANDLER are not define, default values are assigned
```

## ASSUMPTIONS:
  * The associated container can be created given a "CONTEXT" defined as a PATH or URL.
  * Multiple .docker-cgi configuration files can reference the same CONTEXT
  * Given the appropriate .htaccess directives, URLs can be simplified
    * (Refer to Apache directives: DirectoryIndex and Rewrite_Rule)

Revised URL   | Default URL
------------ | -------------
https://www.domain.com/~user/directory/program1 | https://www.domain.com/~user/directory/program1.docker-cgi
https://www.domain.com/~user/directory/program2 | https://www.domain.com/~user/directory/program2.docker-cgi
https://www.domain.com/~user/directory/program3 | https://www.domain.com/~user/directory/program3.docker-cgi

Each of the programs (program1, program2, program3) may use the same container or different containers.


## ENHANCEMENTS:
  * Create a Apache module:
    * define appropriate Apache directives to subsume the role of the config file
    * transform the existing code to allow be used as a Apache Module
    * rewrite the code into C (See: http://linuxdocs.org/HOWTOs/Apache-Overview-HOWTO-12.html)
    *	E.g., the following envisioned Apache directives could be used to to 
    ```
      ContainerEngine off | docker | other    # initial implementation will only be for docker containers
      ContainerCGI   context1 program1			
      ContainerCGI   context2 program2
      ContainerCGI   context1 program3
    ```
  * Examine the potential use of [https://podman.io](podman) as another implementation to support containers
  * Create appropriate man pages for the docker-cgi program and the .docker-cgi file format
  * Consider providing an appropriate "make install" process
  
