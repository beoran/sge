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


config:
  @echo "/* SGE Config header (generated automatically) */" >sge_config.h
  @echo "#define SGE_VER $(SGE_VER)" >>sge_config.h 
ifeq ($(C_COMP),y)
  @echo "#define _SGE_C_AND_CPP" >>sge_config.h
endif
ifeq ($(USE_FT),n)
  @echo "#define _SGE_NOTTF" >>sge_config.h
endif
ifeq ($(USE_IMG),y)
  @echo "#define _SGE_HAVE_IMG" >>sge_config.h
endif
ifeq ($(NO_CLASSES),y)
  @echo "#define _SGE_NO_CLASSES" >>sge_config.h
endif

ifneq ($(QUIET),y)
  @echo "== SGE r$(SGE_VER)"
ifeq ($(C_COMP),y)
  @echo "== Note: Trying to be C friendly."
endif
ifeq ($(USE_FT),n)
  @echo "== FreeType2 support disabled."
else
  @echo "== FreeType2 support enabled."
endif
ifeq ($(USE_IMG),y)
  @echo "== SDL_Image (SFont) support enabled."
else
  @echo "== SDL_Image (SFont) support disabled."
endif
ifeq ($(NO_CLASSES),y)
  @echo "== Warning: No C++ classes will be build!"
endif
  @echo ""  
endif

install:  shared
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
