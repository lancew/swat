# SYNOPSIS

SWAT is Simple Web Application Test ( Tool )

# WHY

I know there are a lot of tests tools and frameworks, but let me  briefly tell _why_ I created swat.
As devops I update a dozens of web application weekly, sometimes I just have _no time_ sitting and wait while dev guys or QA team ensure that deploy if fine
and nothing breaks on the road. So I need a tool to _run smoke tests_ against _web applications_. Not tool only, but way to _create such a tests
from the scratch_ in way easy and fast enough. So this how I came up with the idea of swat. If I was a marketing guy I'd say that swat:

- is easy to use and flexible tool to run smoke tests against web applications
- it's curl powered by and TAP compatible 
- it has minimal dependency tree  and probably will run out of the box on most linux environments, provided that one has perl/bash/find/curl by hand  
( which is true  for most cases )
- it has a simple and yet powerful DSL allow you to both run simple tests ( 200 OK ) or complicated ones ( using curl api )
- and yes ... it's fun (: 

# Tutorial

## Create tests

    mkdir  my-app/ # create a project directory to hold tests


    # define http URI application should response

    mkdir -p my-app/hello # GET /hello-world 
    mkdir -p my-app/hello/world # GET /hello-world 

    # define content the URIs should return

    echo 200 OK >> my-app/hello/get.txt
    echo 200 OK >> my-app/hello/world/get.txt

    echo 'This is hello' >> my-app/hello/get.txt
    echo 'This is hello world' >> my-app/hello/world/get.txt

## Run tests

    swat ./my-app http://127.0.0.1 

# DSL
Swat DSL consists of 2 parts. Routes ( or URIs ) and check patterns.

## Routes

URI resolution is conventional based. It's taken as subtree path taken from project root directory. Let's say we have a project layout:

    example/my-app/    
    example/my-app/hello/    
    example/my-app/hello/get.txt    
    example/my-app/hello/world/get.txt    

When you give swat a run

    swat example/my-app 127.0.0.1 

It just find all the directories holding get.txt files and resolve URI upon project root directory ( which is example/my-app ):

    GET hello/
    GET hello/world

Then check pattern come into play.

## Check patterns 

As you can see from tutorial above check patterns are  just text files describing what you expect to return for given URI. 
Check patterns file parsed by swat line by line and there are 3 types of entities my occur here:

- Expected Values
- Comments
- Perl inline code


### Expected values
This is most usable that one may define at check patterns files. _It's just s string should be returned_ when swat request a given URI. Here are examples:

    200 OK
    Hello World
    <h1><title>Hello World</title></h1>



### Comments
Comments are lines started with '#' symbol, they are for human not for swat which ignore them when parse check pattern file. Here are examples.

    # this http status should return
    200 OK
    Hello World # this string should be in the response 
    <h1><title>Hello World</title></h1> # and it should be html code 


### Inline perl code

Everything started with `code:` would be treated by swat as perl code to execute. 
There are a lot of possibilities. Please follow [Test::More] documentation to get more info about useful function you may call here.

    code: skip('next test is skipped',1) # skip next check forever
    HELLO WORLD

### Using regexp
Regexps are subtypes of expected values, with the only adjustment that you may use _perl regular expressions_ instead of plain strings checks.
Everything started with `regexp:` would be treated as regular expression.

    # this is example of regexp check
    regexp: App Version Number: (\d+\.\d+\.\d+)
    
# Post requests
When talking about swat I always say about Get http request, but swat may send a Post http request just name your check patterns file  as post.txt instead of get.txt 

    echo 200 OK >> my-app/hello/post.txt
    echo 200 OK >> my-app/hello/world/post.txt


# Swat settings

Swat has some settings may redefined as _environmental variables_ and|or using swat.ini files 

## Environmental variables

One may set a proper environment variables to adjust swat settings:

- debug - set to 1 if you want to see some debug information in output, default value is 0
- curl_params - additional curl parameters to add to http requests, default value is ""
- curl_connect_timeout - see curl documentation
- curl_max_time - see curl documentation

## Swat.ini files

Swat also checks files named swat.ini in _every project subdirectory_ and if one exists apply settings from it.
Swat.ini file should be bash file with swat variables definitions:

    # the content of swat.ini file:
    curl_params="-H 'Content-Type: text/html'"
    debug=1

As I say there are many swat.ini files may exist at your project, the one present at the deepest hierarchy level will override predecessors

    my-app/swat.ini
    my-app/hello/get.txt
    my-app/hello/swat.ini 
    my -app/hello-world/get.txt
    my -app/hello-world/swat.ini


# Swat Dependencies
Not that many :)

- perl / curl / bash / find  / head

# AUTHOR
[Aleksei Melezhik](mailto:melezhik@gmail.com)
