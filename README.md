# Training Material for PowerRuby Community Edition v2

## Why a C/C++ compiler with PowerRuby?

When we originally packaged PowerRuby for IBM i PASE we thought there was no need for distributing a compiler with it. We were doing the extra effort of pre-installing relevant gems requiring compilation and we were keeping them  consistent with the shared libraries released in our own packaging. The Ruby standard library was already very rich in functionality for Ruby scripts to be developed...

It is clear now that, apart from compiling public binary gems, a compiler is needed if you have serious plans to develop your own gems leveraging  **C extensions**. 
A gem's extension is usually introduced to wrap libraries that are written in C. Refer to [GEMS WITH EXTENSIONS](https://guides.rubygems.org/gems-with-extensions/) for a good introduction and a *FURTHER READING* paragraph.

Another great tool when working with gems with extensions is [gem-compiler](https://rubygems.org/gems/gem-compiler/versions/0.8.0) gem by Luis Lavena. It will help generating the so-called **binary** gems from existing gems with extensions. 
The result of this process is creating an installable gem that will not require the compiler on the target system. The gem-compiler gem accomplishes  this task without altering the original source code. 

PowerRuby internally adopts gem-compiler to prepare the gems required by Rails that would require a compilation step during a canonical installation process. 


## Listing installed gems

We can list the gems installed by PowerRuby and filter the ones that were compiled through Luis Lavena's gem-compiler (we are using Unix piping and **grep** utility for the filtering):

```console
  $                                                                                                      
> /PowerRuby/prV2R4/bin/gem list | grep powerpc                                                          
  bcrypt (3.1.12 powerpc-aix-7)                                                                          
  bindex (0.5.0 powerpc-aix-7)                                                                           
  bson (4.3.0 powerpc-aix-7)                                                                             
  byebug (10.0.2 powerpc-aix-7)                                                                          
  duktape (2.0.1.0 powerpc-aix-7)                                                                        
  ffi (1.9.25 powerpc-aix-7)                                                                             
  nio4r (2.3.1 powerpc-aix-7)                                                                            
  nokogiri (1.8.4 powerpc-aix-7)                                                                         
  puma (3.12.0 powerpc-aix-7)                                                                            
  sqlite3 (1.3.13 powerpc-aix-7)                                                                         
  websocket-driver (0.6.5 powerpc-aix-7)                                                                 
  $                                                                                                      
```

Now we look (locally) for **gem-compiler** 

```console    
  $                                                                                                       
> /PowerRuby/prV2R4/bin/gem list gem-co                                                                   
                                                                                                          
  *** LOCAL GEMS ***                                                                                      
                                                                                                          
  gem-compiler (0.8.0)                                                                                    
  $  
```

If you just installed PowerRuby it will not be there. If that is the case you have to install it.

## Installing gems in PowerRuby

Be very careful: *Installing gems directly from the Internet is not always the best solution*. 

We suggest to prepare and revise gems by first fetching them on a development system (fetching a gem means downloading it from the gem repository). 

By default this is the remote source for installations:

```console
  $                                             
> /PowerRuby/prV2R4/bin/gem env remotesources   
  https://rubygems.org/                         
  $                                             
```

To install a gem that was pre-fetched (and presumably transferred on the production system):

```console
  $
> /PowerRuby/prV2R4/bin/gem install ./gem-compiler-0.8.0.gem 
  Successfully installed gem-compiler-0.8.0
  Parsing documentation for gem-compiler-0.8.0
  Installing ri documentation for gem-compiler-0.8.0
  Done installing documentation for gem-compiler after 2 seconds
  1 gem installed
  $
```

So let us suppose we just fetched (and transferred) a gem requiring compilation (e.g. [byebug gem](https://rubygems.org/gems/byebug)).                                                                                                  

```console
> /PowerRuby/prV2R4/bin/gem compile ./byebug-10.0.2.gem                  
  Unpacking gem: 'byebug-10.0.2' in temporary directory...
  Building native extensions. This could take a while...
  ERROR:  While executing gem ... (Gem::Ext::BuildError)
      ERROR: Failed to build gem native extension.

      current directory: /tmp/d20181024-341834-u9hxuc/byebug-10.0.2/ext/byebug
  /PowerRuby/prV2R4/bin/ruby -r ./siteconf20181024-341834-nv0s7a.rb extconf.rb
  creating Makefile

  current directory: /tmp/d20181024-341834-u9hxuc/byebug-10.0.2/ext/byebug
  make "DESTDIR=" clean

  current directory: /tmp/d20181024-341834-u9hxuc/byebug-10.0.2/ext/byebug
  make "DESTDIR="
  compiling breakpoint.c
  /bin/sh: gcc:  not found
  make: The error code from the last command is 127.


  Stop.

  make failed, exit code 2

  Gem files will remain installed in /tmp/d20181024-341834-u9hxuc/byebug-10.0.2 for inspection.
  Results logged to /tmp/d20181024-341834-u9hxuc/byebug-10.0.2/lib/gem_make.out
```

Something went wrong but keep calm! The explanation is really simple:

`/bin/sh: gcc:  not found`

We do not have the **gcc** compiler!

## 1PRUBY1 option 2

We have compiled a very recent gcc version for execution in PASE and have packaged it inside option 2 of PowerRuby CE2.

Refer to the official [PowerRuby CE2 Installation Guide](https://github.com/PowerRuby/DE_train_01/blob/master/README.md) for istallation instructions and download the corresponding SAVEFILE [here]().

Once gcc compiler is installed:

| Resource ID | Option | Feature | Description                                    | 
| ----------- |:------ |:-------:|:---------------------------------------------- |   
|   1PRUBY1   |  *BASE |   5001  |  IBM i PowerRuby (administration utilities)    | 
|   1PRUBY1   |  *BASE |   2924  |  IBM i PowerRuby (administration utilities)    |
|   1PRUBY1   |  1     |   5002  |  IBM i PowerRuby Developer Edition (irubydb)   |
|   1PRUBY1   |  2     |   5003  |  IBM i PowerRuby Developer Edition (gcc)       |
|   1PRUBY1   |  6     |   5001  |  IBM i PowerRuby (Ruby 2.4 + Rails 5.1)        |


we can try again our compilation (note the prepended `PATH=/PowerRuby/gcc/bin:$PATH` specification): 

```console
> PATH=/PowerRuby/gcc/bin:$PATH /PowerRuby/prV2R4/bin/gem compile ./byebug-10.0.2.gem                  
  Unpacking gem: 'byebug-10.0.2' in temporary directory...
  Building native extensions. This could take a while...
   /PowerRuby/prV2R4/lib/ruby/site_ruby/2.4.0/rubygems/ext/builder.rb:76: warning: Insecure world writable dir /PowerRuby/gcc/bin in PATH, mode 042777
    Successfully built RubyGem
    Name: byebug
    Version: 10.0.2
    File: byebug-10.0.2-powerpc-aix-7.gem
```

This time the compilation was successful. We have a brand new build of **byebug** for PASE!

Let us install it:

```console
> /PowerRuby/prV2R4/bin/gem install ./byebug-10.0.2-powerpc-aix-7.gem
  Successfully installed byebug-10.0.2-powerpc-aix-7
  Parsing documentation for byebug-10.0.2-powerpc-aix-7
  Installing ri documentation for byebug-10.0.2-powerpc-aix-7
  Done installing documentation for byebug after 35 seconds
  1 gem installed
```
 
