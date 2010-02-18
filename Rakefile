
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
  def ar(options, target, *objects) 
    cmd C[:AR], options, target, objects.join(' ')
  end
  
  def cxx(source, object) 
    cmd C[:CXX], C[:CFLAGS].join(' '), '-c', source, '-o', object  
  end
  
  def cc(source, object) 
    cmd C[:CC], C[:CFLAGS].join(' '), '-c', source, '-o', object
  end
  
  def ld(options, target, *objects)
    cmd C[:LD], options, '-o', target, objects.join(' ')
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
  
  
  def static_library(lib, *objects)
    ar 'rsc', libfile(lib), *objects 
  end
  
  
  def shared_library(lib, objects)
    slib  = slibfile(lib)
    api   = "#{slib}.#{C[:API_VERSION]}"
    ld "-soname #{api} -fpic -fPIC -shared", slib, objects
  end
end

include Kuwa

C[:C_COMP]    = true
C[:USE_FT]    = true
C[:USE_IMG]   = true
C[:QUIET]     = true
C[:AR_NAME]   = 'ar'
C[:CC_NAME]   = 'gcc'
C[:CXX_NAME]  = 'g++'
C[:LD_NAME]   = 'ld'



rule :have_sdl do
 sh "sdl-config --version < /dev/null > /dev/null 2>&1"
end 

# Where should SGE be installed?
C[:PREFIX]    = %x{sdl-config --prefix}.chomp
# Where should the headerfiles be installed?
C[:PREFIX_H]  = "#{C[:PREFIX]}/include/SDL"

# Flags passed to the compiler
C[:CFLAGS]    = ENV['CFLAGS'] ? ENV['CFLAGS'].split(' ') : []
C[:CFLAGS]   += %w{-Wall -O -ffast-math}
# SDL CFLAGS
C[:SDL_CFLAGS]=%x{sdl-config --cflags}.chomp.split(' ')
C[:CFLAGS]   += C[:SDL_CFLAGS]
C[:SDL_LIBS]  = %x{sdl-config --libs}.chomp.split(' ')
C[:LIBS]     << '-lstdc++'

# Is freetype-config available?
C[:HAVE_FT]  =%x{if (freetype-config --version) < /dev/null > /dev/null 2>&1;
then echo "y"; else echo "n"; fi;}.chomp

if C[:HAVE_FT] == 'y' && C[:USE_FT]
  C[:LIBS]   += %x{freetype-config --libs}.chomp.split(' ')
  C[:CFLAGS] += %x{freetype-config --cflags}.chomp.split(' ')
else 
  c[:USE_FT]  = false 
end

# Is SDL_image available?
C[:HAVE_IMG] =%x{if test -e "`sdl-config --prefix`/include/SDL/SDL_image.h"
>/dev/null 2>&1; then echo "y"; else echo "n"; fi;}.chomp

if C[:HAVE_IMG] && C[:USE_IMG]
  C[:LIBS] << '-lSDL_image'
else
  C[:USE_IMG] = false 
end

C[:SGE_VERSION]       = '030809'
C[:API_VERSION]       = 0
C[:PLATFORM_PREFIX]   = '' 
C[:AR]                = C[:PLATFORM_PREFIX] + C[:AR]
C[:CC]                = C[:PLATFORM_PREFIX] + C[:CC]
C[:CXX]               = C[:PLATFORM_PREFIX] + C[:CXX]
C[:LD]                = C[:PLATFORM_PREFIX] + C[:LD]


CPP_FILES             = FileList['*.cpp']
O_FILES               = CPP_FILES.sub(/\.cpp$/, '.o')

C[:OBJECTS]           = %w{sge_surface.o sge_primitives.o sge_tt_text.o
                           sge_bm_text.o sge_misc.o sge_textpp.o sge_blib.o
                           sge_rotation.o sge_collision.o sge_shape.o}

rule :all => [:config, C[:OBJECTS] ] do |r|
  static_library 'SGE', C[:OBJECTS]
end

#Each object depends on thier .cpp and .h file
rule  '.o' => ['.cpp',  '.h']  do |r|
  cxx r.source, r.name
end

rule :shared => :all do |r|
  shared_library 'SGE', C[:OBJECTS]
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
