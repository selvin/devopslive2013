# Python Projects in "twitter commons"

This article shows step by step instructions on how to setup your python projects to build PEXs using pants in [twitter commons](http://github.com/twitter/commons). It uses a bottle HTTP server as the example project.

## Initial Setup

    # Obtain twitter commons
    git clone git@github.com:twitter/commons.git twitter-commons
    
    # Once for a group of projects
    cd twitter-commons/src/python && mkdir -p mydomain && touch mydomain/__init__.py && cd -
    cd twitter-commons/tests/python && mkdir -p mydomain && touch mydomain/__init__.py && cd -

## Project Setup

### Setup Sources

    # Example project -- a bottle server
    cd twitter-commons/src/python/mydomain
    mkdir myproject
    cd myproject

#### Library

    cat >__init__.py <<EOF
    import bottle


    @bottle.route('/')
    @bottle.route('/:name')
    def hello(name='World'):
      return 'Hello %s!\n' % name
    EOF

#### Driver

    mkdir -p bin
    cat >bin/myproject.py <<EOF
    import bottle

    from twitter.common import app

    import mydomain.myproject


    def main():
      bottle.run()


    if __name__ == '__main__':
      app.main()
    EOF

#### BUILD File

    cat >BUILD <<EOF
    python_library(
      name='myproject-lib',
      sources=globs('*.py'),
      dependencies=[
        pants('3rdparty/python:bottle'),
      ],
    )

    python_binary(
      name='myproject',
      dependencies=[
        pants(':myproject-lib'),
        pants('src/python/twitter/common/app'),
      ],
      source='bin/myproject.py'
    )
    EOF

### Build

    ~/workspace/twitter-commons (master)$ ./pants src/python/mydomain/myproject
    Build operating on targets: OrderedSet([PythonBinary(src/python/mydomain/myproject/BUILD:myproject)])
    Building PythonBinary PythonBinary(src/python/mydomain/myproject/BUILD:myproject):
    Wrote /home/selvin/workspace/twitter-commons/dist/myproject.pex

### Run

    ~/workspace/twitter-commons (master)$ dist/myproject.pex 
    Bottle v0.11.6 server starting up (using WSGIRefServer())...
    Listening on http://127.0.0.1:8080/
    Hit Ctrl-C to quit.

### Verify

    $ curl localhost:8080
    Hello World!
    $ curl localhost:8080/devopslive
    Hello devopslive!

    Bottle v0.11.6 server starting up (using WSGIRefServer())...
    Listening on http://127.0.0.1:8080/
    Hit Ctrl-C to quit.

    127.0.0.1 - - [25/Oct/2013 20:42:53] "GET / HTTP/1.1" 200 13
    127.0.0.1 - - [25/Oct/2013 20:42:57] "GET /devopslive HTTP/1.1" 200 18

## Setup Tests

cd twitter-commons/tests/python/mydomain
mkdir myproject
cd myproject

### Write tests

    cat >test_myproject.py <<EOF
    from mydomain.myproject import hello

    def test_hello():
      assert 'Hello World' in hello()
      assert 'Hello devopslive' in hello('devopslive')
    EOF

### Setup BUILD file

    cat >BUILD <<EOF
    python_test_suite(
      name='all',
      dependencies=[
        pants(':test_myproject')
      ]
    )

    python_tests(
      name='test_myproject',
      sources=globs('*.py'),
      dependencies=[
        pants('src/python/mydomain/myproject:myproject-lib')
      ]
    )
    EOF

### Run tests

    ~/workspace/twitter-commons (master)$ ./pants tests/python/mydomain/myproject:all
    Build operating on targets: OrderedSet([PythonTestSuite(tests/python/mydomain/myproject/BUILD:all)])
    ========================================== test session starts ==========================================
    platform linux2 -- Python 2.7.4 -- pytest-2.4.2
    collected 1 items 

    tests/python/mydomain/myproject/test_myproject.py .

    ======================================= 1 passed in 0.04 seconds ========================================
    tests.python.mydomain.myproject.test_myproject                                  .....   SUCCESS


## Existing projects

Continue to maintain setup.py until you are ready to switchover to pants
Add BUILD files to your repo and setup dependencies

Example:

    # Sources are in myproject/src
    ln -s myproject/src twitter-commons/src/python/mydomain/myproject
    cd twitter-commons && ./pants src/python/mydomain/myproject

    # Tests are in myproject/tests
    ln -s myproject/tests twitter-commons/tests/python/mydomain/myproject
    cd twitter-commons && ./pants tests/python/mydomain/myproject:all
