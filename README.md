# Project Name: docker-cgi

# BRIEF DESCRIPTION:
  * an Apache module that allows a user to run one or more CGI programs from within a container.

# PROJECT OBJECTIVES:
  * To provide an Apache handler, as a first step, to manage the associated container, and to appropriate invoke the CGI program
	* To minimize the amount of steps the user needs to perform to configure their environment
	* To define the most simplistic configuration file to derive the most appropriate Apache Directives for the module

# CURRENT STATUS:
  * An Apache handler has been develop that, when trigger, will build, create, and start a docker container, and excute an encompassed CGI program.
  * The script that drives the handler also provides the user with a number of directors to control the process (mostly for debugging purposes)
  * A set of examples provided via the [docker-cgi.examples project](https://github.com/csuntechlab/docker-cgi.examples) 

# PURPOSE:
  * To allow a developer to select their entire programming stack, independent of the main OS.
	* To create an area of isolation between the user's code and the main OS environment.
	* To eliminate the need to deploy the user's container on an independent web-server

# USAGE
```$ ./docker-cgi help ../docker-cgi.examples/cat.docker-cgi
Usage: docker-cgi [DIRECTIVE] FILE
Usage: docker-cgi help

	 DIRECTIVE is an action to be taken. A list of available DIRECTIVEs are provided below
	 FILE	 is the name of a configuration file that used to invoke the Apache Handler
		 This file has the extension of .docker-cgi"Usage: docker-cgi

	 Used as a Apache Handler
	 The following environment variables are expected to be defined:
		 PATH_INFO:		          the URI path of the .docker-cgi file
		 PATH_TRANSLATED:    the file system path of the .docker-cgi file
		 REDIRECT_HANDLER:   the name of the handler: i.e., docker-cgi

Available DIRECTIVEs include:

	 help:	  to provide this usage message
	 build:	  to build the a local image from the CONTEXT
	 rmi:	    to remove the image associated with the CONTEXT
	 build:	  to build the container from the associated image
	 start:	  to start the associated container
	 exec:	  to execute a CGI program defined by the ENTRY variable
	 stop:	  to start the associated container
	 rm:	    to remove the associated container
	 headers: to emit the HTTP response headers defined within FILE
	 envs:	  to emit the environment variables that will be passed to to ENTRY program

For testing purposes, this script can be invoked as if it were being called by the Apache server as follows:

	 $ PATH_TRANSLATED= docker-cgi 

Caveat/Bugs:
	 - When invoked as a CGI Handler from the command line:
	   the PATH_INFO will not be set correctly.
	 - If PATH_INFO or REDIRECT_HANDLER are not define, default values are assigned
```

# ASSUMPTIONS:
  The associated container can be created given a "CONTEXT" defined as a PATH or URL.
  The CGI program is a given ENTRY point within the container.
     (https://docs.docker.com/engine/reference/commandline/build/)

  We assume that only one such CGI program is delivered via the container
  We assume that only one such CGI program is delivered within a given directory

  Given the appropriate .htaccess directives, these assumptions allows the client to invoke the web application via a simple URL, e.g., https:///~/directory/
  (Refer to Apache directives: DirectoryIndex and Rewrite_Rule)

  It is also planned that these assumptions, with further refinement of this project, can be removed.
  Under this refinement, the following URLs are possible:
```
    https:///~/directory/program1
    https:///~/directory/program2
    https:///~/directory/program3
```
  Each of the programs (program1, program2, program3) may use the same container or different containers.

# INSTALLATION NOTES:
  * A web adminstrator places the handler code in a well-known cgi-bin directory, e.g. "/cgi-bin/docker-cgi"
	* The developer places the configuratition, with the well-known extension, e.g., ".docker-cgi", into an appropriate web-folder
	* The developer places the an .htaccess file is said web-folder to invoke the application
    ```
      AddHandler dockerfile .docker-cgi
      Action dockerfile "/cgi-bin/docker-cgi"
    ```
  * A Client can access the code via the URL: https:///~/directory/program.<extention>

# ENHANCEMENTS
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
