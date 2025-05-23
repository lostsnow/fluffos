---
layout: doc
title: parsing / parse_command
---

# parse_command

### NAME

    parse_command() - try to match a string with a given pattern

### SYNOPSIS

    int parse_command( string command, object env|object *oblist,
                       string pattern, mixed arg, ... );

### DESCRIPTION

    parse_command()  is  a piffed up sscanf(3) operating on word basis.  It
    works similar to sscanf(3) in that it takes a pattern  and  a  variable
    set  of  destination  arguments. It is together with sscanf(3) the only
    efun to use pass by reference for other variables  than  arrays.   That
    is, parse_command() returns values in its arguments.

    parse_command()  returns  1  if 'command' is considered to have matched
    'pattern'.

    The 'env' or 'oblist' parameter either holds an object  or  a  list  of
    objects.  If  it holds a single object than a list of objects are auto‐
    matically created by adding the deep_inventory of the object,  ie  this
    is identical:

       parse_command(cmd, environment(), pattern, arg)

    and

       parse_command( cmd, ({ environment() }) +
                      deep_inventory(environment()), pattern, arg)

    'pattern' is a list of words and formats:

       Example string = " 'get' / 'take' %i "
            Syntax:
                    'word'          obligatory text
                    [word]          optional text
                    /               Alternative marker
                    %o              Single item, object
                    %l              Living objects
                    %s              Any text
                    %w              Any word
                    %p              One of a list (prepositions)
                    %i              Any items
                    %d              Number 0- or tx(0-99)

    The  'arg'  list  is zero or more arguments. These are the result vari‐
    ables as in sscanf. Note that one variable is needed for each %_

    The return types of different %_ is:
                    %o      Returns an object
                    %s      Returns a string of words
                    %w      Returns a string of one word
                    %p      Can on entry hold a list of word in array
                            or an empty variable
                            Returns:
                               if empty variable: a string
                               if array: array[0] = matched word
                    %i      Returns a special array on the form:
                            [0] = (int) +(wanted) -(order) 0(all)
                            [1..n] (object) Objectpointers
                    %l      Returns a special array on the form:
                            [0] = (int) +(wanted) -(order) 0(all)
                            [1..n] (object) Objectpointers
                                            These are only living objects.
                    %d      Returns a number

    The only types of % that uses  all  the  loaded  information  from  the
    objects  are %i and %l. These are in fact identical except that %l fil‐
    ters out all nonliving objects from the list of objects  before  trying
    to parse.

    The return values of %i and %l is also the most complex. They return an
    array consisting of first a number and then all possible objects match‐
    ing.   As  the  typical  string matched by %i/%l looks like: 'three red
    roses', 'all nasty bugs' or 'second blue sword'  the  number  indicates
    which of these numerical constructs was matched:

       if numeral >0 then three, four, five etc were matched
       if numeral <0 then second, twentyfirst etc were matched
       if numeral==0 then 'all' or a generic plural form such as
                      'apples' were matched.

### NOTE

    The  efun  makes  no semantic implication on the given numeral. It does
    not matter if 'all apples' or 'second apple' is given. A %i will return
    ALL  possible  objects matching in the array. It is up to the caller to
    decide what 'second' means in a given  context.   Also  when  given  an
    object and not an explicit array of objects the entire recursive inven‐
    tory of the given object is searched. It is up to the caller to  decide
    which  of  the objects are actually visible meaning that 'second' might
    not at all mean the second object in the returned array of objects.

### CAVEAT

    Patterns of type: "%s %w %i" Might not work as one  would  expect.   %w
    will  always  succeed  so  the  arg  corresponding to %s will always be
    empty.

### BUGS

    Patterns of the type: 'word' and [word] The 'word' can not contain spa‐
    ces.   It  must  be  a  single word.  This is so because the pattern is
    exploded on " " (space) and a pattern element can therefore not contain
    spaces.

    As  another effect of the exploding on space, separate pieces of a pat‐
    tern MUST be separated with space, ie not " 'word'/%i " but " 'word'  /
    %i"

### EXAMPLE

         if (parse_command("spray car",environment(this_player()),
                           " 'spray' / 'paint' [paint] %i ",items))
             {
                /*
                  If the pattern matched then items holds a return array as
                  described under 'destargs' %i above.
                */
             }

### MUDLIB SUPPORT

    To make the efun useful it must have a certain support from the mudlib,
    there is a set of functions that it  needs  to  call  to  get  relevant
    information before it can parse in a sensible manner.

    In  earlier versions it used the normal id() lfun in the LPC objects to
    find out if a given object was identified by a certain string. This was
    highly inefficient as it could result in hundreds or maybe thousands of
    calls when very long commands were parsed.

    The new version relies on the LPC objects to give  it  three  lists  of
    'names'.

       1 - The normal singular names.
       2 - The plural forms of the names.
       3 - The acknowledged adjectives of the object.

    These are fetched by calls to the functions:

       1 - string *parse_command_id_list();
       2 - string *parse_command_plural_id_list();
       3 - string *parse_command_adjectiv_id_list();

    The  only really needed list is the first. If the second does not exist
    than the efun will try to create one from the singluar list.  For gram‐
    matical  reasons  it does not always succeed in a perfect way.  This is
    especially true when the 'names' are not single words but phrases.

    The third is very nice to have because it makes  constructs  like  'get
    all the little blue ones' possible.

    Apart  from these functions that should exist in all objects, and which
    are therefore best put in the base mudlib object there is also a set of
    functions needed in the master object.  These are not absolutely neces‐
    sary but they give extra power to the efun.

    Basically these master object lfuns are there to  give  default  values
    for the lists of names fetched from each object.

    The  names  in  these  lists are applicable to any and all objects, the
    first three are identical to the lfuns in the objects:

       string *parse_command_id_list()
          - Would normally return: ({ "one", "thing" })

       string *parse_command_plural_id_list()
          - Would normally return: ({ "ones", "things", "them" })

       string *parse_command_adjectiv_id_list()
          - Would normally return ({ "iffish" })

    The last two are the default list of the prepositions and a  single  so
    called 'all' word.
       string *parse_command_prepos_list()
          - Would normally return: ({ "in", "on", "under" })

       string parse_command_all_word()
          - Would normally return: "all"
