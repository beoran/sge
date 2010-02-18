
if $0 == __FILE__
  require 'rake'
end  

require 'singleton'

# Kuwa (japanese for Hoe) 
# rake helper for compiling and cross compiling C, C++ 
module Kuwa

  class Environment
    attr_accessor :libs
    attr_accessor :cc
    attr_accessor :cxx
    attr_accessor :ar
    attr_accessor :cflags
    
  end 
  

  PATHLIST_SEP = ':'
 
  # Splits a path list from the environment into a Filelist
  def env2filelist(name)
    result = FileList.new
    env    = ENV[name]
    return result unless env
    result << env.split(PATHLIST_SEP)
  end

  def c_libs
    @c_libs ||= env2filelist(name)
    return @c_libs
  end
  
  # Module that helps keeping track of the build environment paths, 
  # constants, etc
  module C
    def self.[](key)
      @kuwa_environment ||= {}
      @kuwa_environment[key]  
    end
    
    def self.[]=(key, *values)
      @kuwa_environment ||= {}
      @kuwa_environment[key.to_sym] = values  
    end
  end  
  
  # Helper functions
  
  def cmd(command, *args)
    aid = command + " " + args.join(" ")
    return sh(aid)
  end
  
  # Call the ar static lib builder
  def ar(options, target, src) 
    cmd(C[:AR], options, target, src.join(' ')) 
  end
  
  def cxx(source, object) 
    sh "#{CXX} #{CFLAGS} -c  #{source} -o #{object}"
  end
  
  def libfile(name)
    return "lib#{name}.a"
  end
  
  def slibfile(name)
    return "lib#{name}.so"
  end
  
  def exefile(name)
    return "#{name}"
  end
  
  
  def static_library(lib, objects)
  sh "#{AR} rsc #{libfile(lib)} #{objects.join(' ')}" 
  end
  
  def shared_library(lib, objects)
  sh "#{CXX} #{CFLAGS} -Wl,-soname,#{slibfile(lib)}.#{API_VER} -fpic -fPIC
  -shared -o #{slibfile(lib)} #{objects.join(' ')} #{LIBS}"
  end

  
end

include Kuwa
C[:LIBS] = 'foo', 'bar'
p C[:LIBS]

# Helper functions

def ar(options, target, src) 
  sh "#{AR} #{options} #{target} #{src.join(' ')}" 
end

def cxx(source, object) 
  sh "#{CXX} #{CFLAGS} -c  #{source} -o #{object}"
end

def libfile(name)
  return "lib{name}.a"
end

def slibfile(name)
  return "lib{name}.so"
end


def static_library(lib, objects)
 sh "#{AR} rsc #{libfile(lib)} #{objects.join(' ')}" 
end

def shared_library(lib, objects)
 sh "#{CXX} #{CFLAGS} -Wl,-soname,#{slibfile(lib)}.#{API_VER} -fpic -fPIC
-shared -o #{slibfile(lib)} #{objects.join(' ')} #{LIBS}"
end


# Configure Makefile for the SGE library


# Comment/uncomment the following line to disable/enable build options
# (See README for more info)
C_COMP  = 'y'
USE_FT  = 'y'
USE_IMG = 'y'
QUIET   = 'n'


# Compilers (C and C++)
CC_SUFIX   = 'gcc'
CXX_SUFFIX = 'g++'

rule :have_sdl do
 sh "sdl-config --version < /dev/null > /dev/null 2>&1"
end 


# Where should SGE be installed?
PREFIX =%x{sdl-config --prefix}

# Where should the headerfiles be installed?
PREFIX_H ="#{PREFIX}/include/SDL"

# Flags passed to the compiler
CFLAGS = '-Wall -O -ffast-math'
SGE_CFLAGS =%x{sdl-config --cflags}
# Uncomment to make some more optimizations

# Libs config
SGE_LIBS =%x{sdl-config --libs} + ' -lstdc++ '

# Is freetype-config available?
HAVE_FT  =%x{if (freetype-config --version) < /dev/null > /dev/null 2>&1;
then echo "y"; else echo "n"; fi;}.chomp

p HAVE_FT

if HAVE_FT == 'y'
  USE_FT    = 'y'
  SGE_LIBS +=%x{freetype-config --libs}
  FT_CFLAGS =%{freetype-config --cflags}
else 
  USE_FT = 'n'
  FT_CFLAGS = ''
end

# Is SDL_image available?
HAVE_IMG =%x{if test -e "`sdl-config --prefix`/include/SDL/SDL_image.h"
>/dev/null 2>&1; then echo "y"; else echo "n"; fi;}

USE_IMG ||= HAVE_IMG

if USE_IMG == 'y'
  SGE_LIBS += '-lSDL_image'
end


CFLAGS_ENV  = ENV['CFLAGS'] || ''
CFLAGS      = CFLAGS_ENV + ' ' + SGE_CFLAGS + ' ' + FT_CFLAGS + ' -fPIC '
LIBS        = SGE_LIBS

SGE_VER = '030809'
API_VER = 0

PLATFORM_PREFIX = ''
AR              = PLATFORM_PREFIX + 'ar'

OBJECTS = %w{sge_surface.o sge_primitives.o sge_tt_text.o sge_bm_text.o
sge_misc.o sge_textpp.o sge_blib.o sge_rotation.o sge_collision.o sge_shape.o}



rule :all => [:config, OBJECTS ] do |r|
  static_library 'SGE', OBJECTS
end

#Each object depends on thier .cpp and .h file
rule  '.o' => ['.cpp',  '.h']  do |r|
  cxx r.source, r.name
end

rule :shared => :all do |r|
  shared_library 'SGE', OBJECTS
end

rule :shared_strip => :shared do |r|
  strip(slibfile('SGE'))
end

require 'rake/clean'
CLEAN.include('*.o')
CLEAN.include('*.so')
CLEAN.include('*.so.*')
CLEAN.include('*.dll')
CLEAN.include('*.def')


rule :config do
  cfg = File.open('sge_config.h')
  cfg.puts "/* SGE Config header (generated automatically) */" 
  cfg.puts "#define SGE_VER #{SGE_VER}" 
  if C_COMP == 'y'
    cfg.puts "#define _SGE_C_AND_CPP"
  end
  if USE_FT == 'n'
    cfg.puts "#define _SGE_NOTTF"
  end
  if USE_IMG == 'y'
    cfg.puts "#define _SGE_HAVE_IMG"
  end
  if NO_CLASSES == 'y'
    cfg.puts "#define _SGE_NO_CLASSES"
  end
end

rule :install => :shared do
=begin
  @mkdir -p $(PREFIX_H)
  install -c -m 644 sge*.h $(PREFIX_H)
  @mkdir -p $(PREFIX)/lib
  install -c -m 644 libSGE.a $(PREFIX)/lib
  install -c libSGE.so $(PREFIX)/lib/libSGE.so.$(API_VER).$(SGE_VER)
  @cd $(PREFIX)/lib;\
  ln -sf libSGE.so.$(API_VER).$(SGE_VER) libSGE.so.$(API_VER);\
  ln -sf libSGE.so.$(API_VER) libSGE.so
  @echo "** Headerfiles installed in $(PREFIX_H)"
  @echo "** Library files installed in $(PREFIX)/lib"
=end
end

=begin

require 'rake/clean'
require 'rake/loaders/makefile'

APPLICATION = 'test.exe'
CPP_FILES = FileList['*.cpp']
O_FILES = CPP_FILES.sub(/\.cpp$/, '.o')

# Dependencies
file '.depend.mf' do
  sh "makedepend -f- --  -- #{CPP_FILES} > .depend.mf"
end

import ".depend.mf"

file APPLICATION => O_FILES do |t|
  sh "g++ #{O_FILES} -o #{t.name}"
end

rule ".o" => [".cpp"] do |t|
  sh "g++ -c -o #{t.name} #{t.source}"
end

CPP_FILES.each do |src|
  file src.ext(".o") => src
end

CLEAN.include("**/*.o")
CLEAN.include(APPLICATION)
CLEAN.include(".depend.mf")

task :default => APPLICATION
=end
