
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
PREFIX =$(shell sdl-config --prefix)

# Where should the headerfiles be installed?
PREFIX_H =$(shell sdl-config --prefix)/include/SDL

# Flags passed to the compiler
CFLAGS = '-Wall -ffast-math'
SGE_CFLAGS =$(shell sdl-config --cflags)
# Uncomment to make some more optimizations
#CFLAGS =-Wall -O9 -ffast-math -march=i686


# Libs config
SGE_LIBS =$(shell sdl-config --libs) -lstdc++


# Is freetype-config available?
HAVE_FT =$(shell if (freetype-config --version) < /dev/null > /dev/null 2>&1;
then echo "y"; else echo "n"; fi;)
ifeq ($(HAVE_FT),n)
  USE:_FT = n
endif

ifneq ($(USE_FT),n)
  USE_FT = y
  SGE_LIBS +=$(shell freetype-config --libs)
  FT_CFLAGS =$(shell freetype-config --cflags)
endif


# Is SDL_image available?
HAVE_IMG =$(shell if test -e "`sdl-config --prefix`/include/SDL/SDL_image.h"
>/dev/null 2>&1; then echo "y"; else echo "n"; fi;)

ifneq ($(USE_IMG),y)
  ifneq ($(USE_IMG),n)
    USE_IMG =$(HAVE_IMG)
  endif
endif

ifeq ($(USE_IMG),y)
  SGE_LIBS +=-lSDL_image
endif


CFLAGS  = ENV['CFLAGS'] || ''
CFLAGS += SGE_CFLAGS + FT_CFLAGS + ' -fPIC '
LIBS    = SGE_LIBS

SGE_VER = '030809'
API_VER = 0

PLATFORM_PREFIX = ''
AR              = PLATFORM_PREFIX + 'ar'

OBJECTS = %w{sge_surface.o sge_primitives.o sge_tt_text.o sge_bm_text.o
sge_misc.o sge_textpp.o sge_blib.o sge_rotation.o sge_collision.o sge_shape.o}


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