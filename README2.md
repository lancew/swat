# SYNOPSIS

Web automated testing framework.

# Description

- Swat is a powerful and yet simple and flexible tool for rapid web automated testing development.
- Swat is a web applicaton oriented test framework, this means that it equipes you with nothing more than you need
to automatcaly test your web application, it is light weighted  and easy to use tool not burdened by many other "generic" things that you probably won't ever use.
- Swat does not carry all heavy load on it's shoulder, with the help of it's "older brother" - curl
swat makes a http requests in a smart way. This means if you know and love curl swat might be easy way to go.
Swat just passes all curl related parameter as is to curl and let curl do it's job.
- Swat is text oriented tool, for good or for bad it does not provide any level of http DOM or xpath hacking, it does
not even try to decouple http headers from a body. Actually _it just returns you a text_ where you can find and grep
in old good unix way. Does this sound suspiseosly simple? Sometimes most of things could be tested in a simple way.
- Swat is extendable by adding custom perl code, this is where you may add deisred complexity to your test stories.
- And finally swat relies on prove as internal test runner, this has many, many good results:
    - swat transparently pass all it's arguments to prove which make it simple to adjust swat runner behavior in a prove way
    - swat tests might be easily embedded as unit tests into a cpan distributions.
    - test reports are emitted in a TAP format which is portable and easy to read.

Ok, now I hope you are ready to dive into swat tutorial! :)

# Install

    $ sudo apt-get install curl
    $ sudo cpanm swat

Or install from source:

    # useful for contributors and developers
    perl Makefile.PL
    make
    make test
    make install

# Building up swat test harness

Swat test stories always answer on 2 type of questions:

- _What kind of_ http request should be send
- _What kind of_ http response should be recieved

As swat is web test oriented tool it deals with some http related stuff as 

- http methods
- http resourses 
- http responses

Swat leverages unix file system to build an _analogy_ for these things:

## HTTP Resources

_HTTP resourse is just a directory_. You have to create a directory to define a http resourse:

    mkdir foo/
    mkdir -p bar/baz

This code define two http resourses of a tested application - 'foo/' and 'bar/baz'

## HTTP methods

_HTTP method is just a file_. You have to create a file to define a http method.

    touch foo/get.txt
    touch foo/put.txt
    touch bar/baz/post.txt

It's obvious that \`http methods' files should be located at \`http resourse' directories.

All the code above defines a three http methods for two http resources:

    - GET /foo
    - PUT /foo
    - POST bar/baz

Here is the list of _predifened_ file names for a http methods files:

    get.txt --> GET method
    post.txt --> POST method
    put.txt --> PUT method
    delete.txt --> DELETE method

# Hostname / IP Address

You need to define hostname or ip address of the application to send request to. The easiest way to do this
is to write hostname or ip address to a file. Swat looks for a file named \`host' to get know web application address:

    echo 'app.local' > host

As swat uses curl to make http requests, the host name just need to be compilant with curl requrements, this
for exmaple means you may define a http schema or port here:

    echo 'https://app.local' >> host
    echo 'app.local:8080' >> host

## HTTP Response

Swat makes request to a given http resourses with a given http methods and then validates repsonse.
Swat does this with the help so called _check lists_ defined at http method files.

Check list is just a list of strings a response should match. It might be plain string or regular expressions:

    echo 200 OK > foo/get.txt
    echo 'Hello I am foo' >> foo/get.txt

The code above defines two test asserts for response returned from \`GET /foo':

    - it should contain "200 OK"
    - it should contain "Hello I am foo"

Of cousre as I told you may add some regular expressions checks here:

    echo 'regexp: \d\d\d\d-\d\d-\d\d' >> foo/get.txt

# Swat stories

Bringing all together we have a basic entity for swat test harness - a _swat story_.

Swat story is a very simple test plan, which could be expressed in a cucumber language as follows:

    Given I have web application 'http://my.cool.app:80'
    And I have http method 'GET'
    And make http request 'GET /foo'
    Then I should have response matches '200 OK'
    And I should have response matches 'Hello I am foo'
    And I should have response matches '\d\d\d\d-\d\d-\d\d'

From other hand a swat stories is always 3 related things:

- http method - the method file
- http resourse - the directory where \`method file\` located in
- check list - the content of method file

## Swat Project

Swat project is sereval related swat stories kept under a single directory. It does not realy matter the name of such a directory,
swat just looks up swat stories files there and then "execute" them ( see ["Swat to Test::Harness Compilation"](#swat-to-test-harness-compilation) section on how swat executes test stories ).

This is example of a swat project:

    $ tree my/swat/project
    my/swat/project
    |--- host
    |----FOO
    |-----|----BAR
    |           |---- post.txt
    |--- FOO
          |--- get.txt

    3 directories, 3 files

When you ask swat to run swat stories you need to point it a project root directory or \`cd' to it and just run swat without arguments:

    swat my/swat/project

    # or

    cd my/swat/project && swat

Note, that project root directory path will be removed from pathes for http resourses during execution:

    - GET FOO
    - POST FOO/BAR

It is possible to run a subset of swat test stories using a `test_file` variable:

    # run a single test
    test_file=FOO/get swat example/my-app 127.0.0.1

    # run all `FOO/*' stories:
    test_file=FOO/ swat example/my-app 127.0.0.1

Now lets go for swat DSL for describing check lists.

# Check lists

Swat parses http response and determine if it matches to the expressions from the check list:

    # http response
    200 OK
    HELLO
    HELLO WORLD
    My birth day is: 1977-04-16


    # check list
    200 OK
    HELLO
    regexp: \d\d\d\d-\d\d-\d\d


    # swat output
    200 OK matches
    HELLO matches
    regexp: \d\d\d\d-\d\d-\d\d matches

Note, that swat does not care when more than one lines of response matches to a single expression from check list.
It just ok if only one line from respose matches check list experssion. However it is possible to accumulate all matching lines for further processing, see the ["captures"](#captures) section.

Another important thing about check lists is that internally they are represented as Test::More asserts:

# Swat to Test::Harness compilation

Swat parses swat stories and creates a Test::Harness files to be executed recursively by the prove.

Let we have 3 swat stories:

    user/get.txt # GET /user
    user/post.txt # POST /user
    users/list/get.txt # GET /users/list

As the result of compilation there are 3 Test::Harness test files here:

    user/get.t
    user/post.t
    users/get.txt

The swat check list for every swat story is converted into the list of the Test::More asserts:

    # cat user/get.txt

    200 OK
    regexp: name: \w+
    regexp: age: \d+

    # cat user/get.t

    SKIP {
        ok($status,'response matches 200 OK'); # will pass if response includes string '200 OK'
        ok($status,'response matches name: \w+'); # will pass if response has strings matched to /name: \w+/ regexp
        ok($status,'response matches age: \d+'); # etc
    }

Thus swat stories execution consists of two parts:

- **Compilation phase** where swat stories are converted into Test::Harness format.
- **Execution phase** where test harness tests are executed by prove.

Let's get back to DSL for defining check list. Every item of check list is _string_ might represent different things:

- **plain string**

        200 OK
        HELLO SWAT
        

    This just ask swat to check if http response has a lines matches to '200 OK' and 'HELLO SWAT' strings.

- **regular expression**

    Similar to plain strings, you may ask swat to check if http response has a lines matches to a regular expressions.

        regexp: \d\d\d\d-\d\d-\d\d # date in format of YYYY-MM-DD
        regexp: 20\d # successful http status 200, 201 etc
        regexp: (red|green|blue) # one of three colors
        regexp: App Version Number: \d+\.\d+\.\d+ # version number

    Regular expression should start with `regexp:` marker.
    You may use `(`,`)` to capture subparts of matching strings, the captured chunks will be saved and could be used further, see ["captures"](#captures) section for this.

        regexp: Hello, my name is (\w+)

- **comments**

    Comment lines start with `#` symbol, swat ignore comments chunks when parse swat stories

        # comments could be represented at a distinct line, like here
        200 OK
        Hello World # or could be added to existed matchers to the left, like here

- **blank lines**

    Blank lines found are ignored. You may use blank lines to improve code readability:

        # check http header
        200 OK
        # then 2 blank lines


        # then another check
        HELLO WORLD

    ... But you **can't ignore** blank lines in a `text block matching` context ( see next point ), use `:blank_line` marker to match blank lines:

        # :blank_line marker matches blank lines
        # this is especially useful
        # when match in text blocks context:

        begin:
            this line followed by 2 blank lines
            :blank_line
            :blank_line
        end:

- **matching text blocks**

    Sometimes it is very helpful to match a response against a `block of strings` goes consequentially, like here:

        # this text block
        # consists of 5 strings
        # goes consequentially
        # line by line:

        begin:
            # plain strings
            this string followed by
            that string followed by
            another one
            # regexps patterns:
        regexp: with (this|that)
            # and the last one in a block
            at the very end
        end:

    This test will pass when running against this chunk:

        this string followed by
        that string followed by
        another one string
        with that string
        at the very end.

    But **won't** pass for this chunk:

        that string followed by
        this string followed by
        another one string
        with that string
        at the very end.

    `begin:` `end:` markers decorate \`text blocks\` content. `:being|:end` markers should not be followed by any text at the same line.

    Also be aware if you leave "dangling" `begin:` marker without closing `end`: somewhere else swat will remain in a \`text block\` mode 
    till the end of your swat story, which is probably not you want:

        begin:
        here we begin
        and till the very end of test
        we are in `text block` mode

- **perl expressions**

    Perl expressions are just a pieces of perl code to _get evaled_ inside your swat story. This is how it works:

        # this is my swat story
        200 OK
        code: print "hello world"
        That's OK

    The piece of code above will be processed in two phases according to ["Swat to Test::Harness Compilation"](#swat-to-test-harness-compilation) specification:

    First swat converts swat story into Test::Harness test, and adds eval "{code}" line into it:

        ok($status,"response matches 200 OK");
        eval 'print "hello world"';
        ok($status,"content matches That's OK"); # etc

    Then prove execute a generated code with a eval expression.

    The example with 'print "hello world"' is quite meanignless, there are of course more effective ways how you code use perl expressions in your swat stories.

    One of the obvious way to use swat perl expressions is too call Test::More functions to adjust execution phase logic:
    ( dependency of Test::More module is already here and need not to be \`used' )

        # skip tests
        code: skip('next 3 checks are skipped',3) # skip three next checks forever
        color: red
        color: blue
        color: green

        number:one
        number:two
        number:three

        # skip tests under conditions

        color: red
        color: blue
        color: green

        code: skip('numbers checks are skipped',3)  if $ENV{'skip_numbers'} # skip three next checks if skip_numbers set 

        number:one
        number:two
        number:three

    As you may noticed perl expressions are executed in a _string eval_ manner, please be aware of this.
    Follow [http://perldoc.perl.org/functions/eval.html](http://perldoc.perl.org/functions/eval.html) to get know about perl eval function restrictions.

- **generators**

    Swat generators is the way to _create swat check lists  on the fly_. Swat generators like perl expressions is just a piece of perl code 
    executed the same way. The only difference with perl expressions is that swat generators code should return _an array reference_.
    An array returned by generator code should contain _strings_ representing new check list items. Thus new check list will passed back to swat
    parser for dynamic check list generation. Here is a simple exmaple:

        # this is `static' check list
        200 OK
        HELLO
        
        # and this is simple swat generator
        # to append new items 
        # to the check list
        generator: [ qw{ foo bar baz } ]


        # the resulted check list will be
        200 OK
        HELLO
        foo
        bar
        baz

    Generators expressions start with `:generator` marker. Here is more example:

        # you could you any perl constrution in generator code
        # unless it return an array reference
        generator: my %d = { 'foo' => 'foo value', 'bar' => 'bar value' }; [ map  { ( "# $_", "$data{$_}" )  } keys %d  ]


        # the resulted check list will be:

        # foo
        foo value
        # bar
        bar value

    Writting generator code there is no limit for you! Use any code you want with only requirement - it should return array reference.

    What about to validate web application content with sqlite database entries?

        generator:                                                          \

        use DBI;                                                            \
        my $dbh = DBI->connect("dbi:SQLite:dbname=t/data/test.db","","");   \
        my $sth = $dbh->prepare("SELECT name from users");                  \
        $sth->execute();                                                    \
        my $results = $sth->fetchall_arrayref;                              \

        [ map { $_->[0] } @${results} ]

    Note about **PERL5LIB**.

    Swat adds **$project\_root\_directory/lib** path to PERL5LIB, so this is convenient to place here custom perl modules could be used inside swat stories:

        my-app/lib/Foo/Bar/Baz.pm

        # now it is possible to use Foo::Bar::Baz
        code: use Foo::Bar::Baz; # etc ...

- **multiline expressions**

    As long as swat deals with matching expressions ( both plain strings or regular expressions ) it works in a single line mode, that means
    it does not make a sense to tell about multilne strings here:

           # swat story
           Yet another 
           new string here
               

           # http response
           Yet another\nstring here
           
        
           # swat put output

           Yet another - matched by "Yet another"
           new string here - matched by "tring here"

    Often there is no need to operate on multiline string mode, as with the help of text blocks it is possible to express very
    complicated matching expressions.

    However as long as talk about perl expressions and generators it is convininet to use multiline code here. It is possible with a `\` delimiters:

        # this is a generator
        # with multiline code
        generator:                  \
        my %d = {                   \
            'foo' => 'foo value',   \
            'bar' => 'bar value',   \
            'baz' => 'baz value'    \
        };                          \
        [                                               \
            map  { ( "# $_", "$data{$_}" )  } keys %d   \
        ]                                               

# Captures

Captures are pieces of data get captured when swat matches response against regular expressions:

    # here is response data.
    # it is just my family ages.
    alex    38
    julia   25
    jan     2


    # here is check list
    # with regular experssion check
    regexp: /(\w+)\s+(\d+)/

_After_ swat execute last regular expression check it captured _all found_ sub parts and stored into array:

    [
        ['alex',    38 ]
        ['julia',   32 ]
        ['jan',     2  ]
    ]

Now captures might be accessed by code generators to define some extra checks:

    code:                               \
    my $total=0;                        \
    for my $c (@{captures()}) {         \
        $total+=$c->[0];                \
    }                                   \
    cmp_ok( $total,'==',72,"total age of my family" );

Thus perl expressions and code generators access captures data calling `captures()` function.

Captures() returns an array reference holding all data captured during _latest regular expression check_.

Here some more examples:

    # check if response contains numbers
    # calculate total amount
    # it should be greater then ten

    regexp: (\d+)
    code:                               \
    my $total=0;                        \
    for my $c (@{captures()}) {         \
        $total+=$c->[0];                \
    }                                   \
    cmp_ok( $total,'>',10,"total amount is greater than 10" );


    # check if response contains lines
    # with date formatted as `date: YYYY-MM-DD`
    # check if first date found is yesterday

    regexp: date: (\d\d\d\d)-(\d\d)-(\d\d)
    code:                               \
    use DateTime;                       \
    my $c = captures()->[0];            \
    my $dt = DateTime->new( year => $c->[0], month => $c->[1], day => $c->[2]  ); \
    my $yesterday = DateTime->now->subtract( days =>  1 );     \
    cmp_ok( DateTime->compare($dt, $yesterday),'==',0,"first day found is - $dt and this is a yesterday" );

You also may use `capture()` function to get a _first element_ of captures array:

    # check if response contains numbers
    # a first number should be greater then ten

    regexp: (\d+)
    code: cmp_ok( capture()->[0],'>',10,"first number is greater than 10" );

# Swat ini files

Every swat story comes with some settings you may define to alter story execution. One of the usual cases is to add http data when making POST or PUT requests.

These type of settings could be defined at swat ini files.

Swat ini files are file called "swat.ini" and located at resources directory:

     foo/bar/get.txt
     foo/bar/swat.ini

The content of swat ini file is the list of variables definitions in bash format:

    $name=value

All swat variables could be devided on two groups:

- **common swat settings**
- **http parameters**

## common swat settings

Common swat settings is a way to adjust _common_ swat behaviour/output.

Here is the list of such varibles which brief explanation:

- `debug` - set to `1,2` if you want to see some debug information in output, default value is `0`.
- `debug_bytes` - number of bytes of http response  to be dumped out when debug is on. default value is `500`.
- `swat_debug` - run swat in debug mode, default value is `0`.
- `ignore_http_err` - ignore http errors, if this parameters is set to \`1' swat does not raise a test failure when 
unsuccessfull http codes (! 2\*\*,3\*\* ) return. Default value is `0`. 
- `prove_options` - prove options to be passed to prove runner,  default value is `-v`.

## http parameters

Setting  http paramters alter http request logic, most of these parameters are refered to curl.

- `try_num` - number of http requests attempts in case of none successful http code return, default value is `2`.
- `curl_params` - additional curl parameters being add to http requests, default value is `""`, follow curl documentation for variety of values for this.
These are some examples:
    - `-d` - Post data sending as html form submit.

             curl_params='-d name=daniel -d skill=lousy'

    - `--data-binary` - Post data sending as is.

             curl_params=`echo -E "--data-binary '{\"name\":\"alex\",\"last_name\":\"melezhik\"}'"`
             curl_params="${curl_params} -H 'Content-Type: application/json'"
- `curl_connect_timeout` - follow curl documentation
- `curl_max_time` - follow curl documentation
- `port`  - http port of tested host, default value is `80`

## Alternative swat ini files locations

Similiary to resourse based swat.ini files you may have swat settings at these locations:

- **~/swat.ini** - home directory settings
- **$project\_root\_directory/swat.ini** -  project based settings
- **$cwd/swat.my** - custom settings

## Settings priority table

This table describes all possible locations for swat ini files. Swat applies settings from
files in order, so settings defined at last found ini files wins.

    | location                               | order N     |
    | ---------------------------------------------------- |
    | ~/swat.ini                             | 1           |
    | project_root_directory/swat.ini        | 2           | 
    | http resourse directrory/swat.ini file | 3           |
    | curent working direcroy/swat.my  file  | 4           |
    | environment variables                  | 5           |

# Hooks

Hooks are extension points you may implement to hack into swat compile / runtime workflow. There are two types of hooks:

- Perl hooks
- Bash Hooks

## Perl hooks

Perl hooks are files with perl code \`required\` _in the beginning/end of a swat test_. There are four types of perl hooks:

- **project based perl startup hook**

    File located at `$project_root_directory/hook.pm`.

    Project based startup hooks are \`required\` _in the beginning_ of a swat test and applied for every route in project
    and thus could be used for _project initialization_ procedures.

    For example one could define common generators here:

        # place this in hook.pm file:
        sub list1 { | %w{ foo bar baz } | }
        sub list2 { | %w{ red green blue } | }


        # now we could use it in swat data file
        generator:  list()
        generator:  list2()

- **project based perl cleanup hook**

    File located at `$project_root_directory/cleanup.pm`.

    This hooks is similar to startup hook but \`required\` _in the end_ of a swat test.

- **route based perl startup hooks**

    Files located at `$route_directory/hook.pm`.

    Routes based startup hooks are applied for every route in project and thus could be used for _route initialization_ procedures.

    For example one could define route specific generators here:

        # place this in hook.pm file:
        # notices that we could tell GET from POST http methods here
        # using predefined $method variable

        sub list1 {

            my $list;

            if ($method eq 'GET') {
                $list = | %w{ GET_foo GET_bar GET_baz } |
            }elsif($method eq 'POST'){
                $list = | %w{ POST_foo POST_bar POST_baz } |
            }else{
                die "method $method is not supported"
            }
            $list;
        }


        # now we could use it in swat data file
        generator:  list()

- **route based perl cleanup hooks**

    Files located at `$route_directory/cleanup.pm`.

    This hooks is similar to route based startup hooks but \`required\` _in the end_ of a swat test.

## Bash hooks

Similar to perl hooks bash hooks are just a bash files \`sourced\` _before compilation_ of a swat test.

There are 4 types of bash hooks:

- **project based bash hook**

    File located at `$project_root_directory/hook.bash`.

    Project based bash hooks are applied for every route in project and could be used for _project initialization_ procedures.

- **route based bash hooks**

    Files located at `$project_root_directory/$route_directory/hook.bash`.

    Routes based bash hooks are route specific hooks and could be used for _route initialization_ procedures.

- **global startup bash hook**

    File located at `$project_root_directory/startup.bash`.

    Startup hook is executed before swat tests gets compiled, at the very beginning, at could be used for _global initialization_ procedures.

- **global cleanup bash hook**

    File located at `$project_root_directory/cleanup.bash`.

    Cleanup hook is executed _after swat tests are executed_, at the very end, and could be used for _global cleanup_ procedures.

It is important to note that bash hooks are executed _after swat settings merge done_ , see  ["Swat Settings"](#swat-settings) section to get more
about swat settings.

## Predefined variables

List of variables one may rely upon when writing perl/bash hooks:

- **http\_url**
- **curl\_params**
- **http\_meth** - `GET|POST|HEAD`
- **route\_dir**
- **project**

# Swat Compile and Runtime

    - Execute *global startup bash hook*
    - Start of swat compilation phase
    - For every swat story found:
       -- Merge swat settings
        -- Set predefined variables
        -- Execute *project based bash hook*
        -- Execute *route based bash hook*
        -- Compile swat story into Test::Harness test
    - The end of swat compilation phase
    - Start of swat execution phase.
    - For every Test::Harness gets executed:
        -- Execute *project based perl startup hook*
        -- Execute *route based perl startup hook*
        -- Execute route test
        -- Execute *route based perl cleanup hook*
        -- Execute *project based perl cleanup hook*
    - The end of swat compilation phase
    - Execute *global cleanup bash hook*

# TAP

Swat produces output in [TAP](https://testanything.org/) format , that means you may use your favorite tap parsers to bring result to
another test / reporting systems, follow TAP documentation to get more on this. Here is example for converting swat tests into JUNIT format

    swat <project_root> <host> --formatter TAP::Formatter::JUnit

See also ["Prove settings"](#prove-settings) section.

# Command line tool

Swat is shipped as cpan package, once it's installed ( see ["Install"](#install) section ) you have a command line tool called **swat**, this is usage info on it:

    swat <project_root_dir|swat_package> <host:port> <prove settings>

- **host** - is base url for web application you run tests against, you also have to define swat routes, see DSL section.
- **project\_dir** - is a project root directory
- **swat\_package** - the name of swat package, see ["Swat Packages"](#swat-packages) section

## Default Host

Sometimes it is helpful to not setup host as command line parameter but define it at $project\_root/host file. For example:

    # let's create a default host for foo/bar project

    $ cat foo/bar/host
    foo.bar.com

    $ swat foo/bar/ # will run tests for foo.bar.com

## Debugging

set `swat_debug` environment variable to 1

## Running a subset of tests

It is possible to run a subset of swat test setting a `test_file` variable:

`test_file`={unix file path} . Test\_file path might be relative or absolute unix file path, internally swat just try to find all proper files using a trivial `unix find`
construction:

    find $test_file -name  -type f -name get.txt -o -name post.txt -o -name put.txt

## Prove settings

Swat utilize [prove utility](http://search.cpan.org/perldoc?prove) to run tests, so all the swat options _are passed as is to prove utility_.
Follow [prove](http://search.cpan.org/perldoc?prove) utility documentation for variety of values you may set here.
Default value for prove options is  `-v`. Here is another examples:

- `-q -s` -  run tests in random and quite mode

# Swat Packages

Swat packages is portable archives of swat tests. It's easy to create your own swat packages and share with other.

This is mini how-to on creating swat packages:

## Create swat package

Swat packages are _just cpan modules_. So all you need is to create cpan module distribution archive and upload it to CPAN.

The only requirement for installer is that swat data files should be installed into _cpan module directory_ at the end of install process.
[File::ShareDir::Install](http://search.cpan.org/perldoc?File%3A%3AShareDir%3A%3AInstall) allows you to install
read-only data files from a distribution and considered as best practice for such a things.

Here is example of Makefile.PL for [swat::mongodb package](https://github.com/melezhik/swat-packages/tree/master/mongodb-http):

    use inc::Module::Install;

    # Define metadata
    name           'swat-mongodb';
    all_from       'lib/swat/mongodb.pm';

    # Specific dependencies
    requires       'swat'         => '0.1.28';
    test_requires  'Test::More'   => '0';

    install_share  'module' => 'swat::mongodb', 'share';

    license 'perl';

    WriteAll;

Here we create a swat package swat::mongodb with swat data files kept in the project\_root directory ./share and get installed into
`auto/share/module/swat-mongodb` directory.

Once we uploaded a module to CPAN repository we can use it:

    $ cpan install swat::mongodb
    $ swat swat::mongodb 127.0.0.1:28017

Check out existed swat packages here - https://github.com/melezhik/swat-packages/

# Examples

./examples directory contains examples of swat tests for different cases. Follow README.md files for details.

# AUTHOR

[Aleksei Melezhik](mailto:melezhik@gmail.com)

# Home Page

https://github.com/melezhik/swat

# Thanks

To the authors of ( see list ) without who swat would not appear to light

- perl
- curl
- TAP
- Test::More
- prove

# COPYRIGHT

Copyright 2015 Alexey Melezhik.

This program is free software; you can redistribute it and/or modify it under the same terms as Perl itself.