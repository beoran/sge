# Makefile for the SGE library examples

CFLAGS = `sdl-config --cflags`
LIBS = `sdl-config --libs` -lSGE

TARGETS = fire bitmapfont collision blib rotate speedtest sprite sprite2 poly basics blitting input alpha inputdeluxe sfont
C_TARGETS = basics_c

OBJECTS = $(addsuffix .o, $(TARGETS))
C_OBJECTS = $(addsuffix .o, $(C_TARGETS))


all:	$(TARGETS) $(C_TARGETS) 

$(TARGETS):	%:%.o
	$(CXX) -o $@ $< $(LIBS)		

$(OBJECTS):	%.o:%.cpp
	$(CXX) $(CFLAGS) -c $< 	

strip:	all
	@strip $(TARGETS)

# The c example
$(C_TARGETS):	%:%.o
	$(CC) -o $@ $< $(LIBS)

$(C_OBJECTS):	%.o:%.c
	$(CC) $(CFLAGS) -c $<


clean:
	@rm -f *.o fire bitmapfont collision blib rotate basics blitting input alpha sfont sprite sprite2 inputdeluxe basics_c speedtest poly
