# TextFinder

> This is a GitHub repository hosting a copy of the [TextFinder](http://playground.arduino.cc/Code/TextFinder) Arduino library with original copyright (c) 2009 2010 Michael Margolis.


TextFinder is a library for extracting information from a stream of data. It was created to be used with the Arduino Ethernet library to find particular fields and get strings or numeric values. I can also be used with Serial data.

Note that parsing functionality based on this library is now included in the Arduino 1.0  core. You can still this library with 1.0 but the core Stream parsing functionality in 1.0 is easier to use because you don’t have to create an instance of the TextFinder object in your sketch.  If you want to use core parsing available in 1.0, see the  Stream documentation for details as  some of the method  names have changed from the standalone TextFinder library.

To use TextFinder, you create an ‘instance’ for the stream you want to processes. For example, the following fragment is based on the Ethernet client example sketch distributed with Arduino. That sketch does a google search on the term arduino.  The following adds functionality to print the number of hits reported by google.

Add these lines  before setup:

    #include <TextFinder.h>
    TextFinder finder( client );

Modify the connection code as follows:

    if (client.connected())
    {
      finder.find("Results <b>");  // seek to the Results field
      finder.find("of about <b>"); // skip past this
      long value = finder.getValue(); // get numeric value
      Serial.print(value);
      Serial.println(" hits");
    }

The remainder of the sketch can be as in the distributed example. The find method reads data from the client until it finds the string: "Results \<b\>"
It then looks for the string: "of about \<b\>"
And the getValue method returns the next numeric value.

The parsing is done without using a buffer so there is no way to go back through data that has already been read.

Here are the supported methods:

    TextFinder( Client &stream, int timeout = 5); // ethernet
    TextFinder(HardwareSerial &stream, int timeout = 5); // Serial

The constructor is given the client stream instance to process. The timeout is the number of seconds to wait for the next character before aborting the find and get methods. The default is 5 seconds if no timeout is given.

    boolean find(char *target);

This reads from the stream until the given target is found. It returns true if the target string is found. A return of false means the data has not been found anywhere in the stream and that there is no more data available. Note that TextFinder takes a single pass through the stream,  there is no way to go back to try and find or get something else (see the findUntil method ).

    boolean findUntil(char *target, char *terminate);

As above but the search will stop if the terminate string is found. Returns true only if target is found.
This is useful to stop a search on a keyword or terminator. For example,
`finder.findUntil("value", "\n\r")"`
will try to seek to the string “value” but will stop at the first blank line (a newline followed immediately by a carriage return).  This enables a search for another term to proceed from the place the terminator was found.

    long getValue();

Returns first valid (long) integer value. Leading characters that are not digits or minus sign are skipped. The integer is terminated by the first non-digit character (other than commas, which are discarded)following the number. If no digits are found the function returns 0.

    long getValue(char skipChar);

As getValue above but the given skipChar is ignored. This allows format characters (for example commas in values) to be ignored.

    float getFloat();

float version of getValue

    float getFloat(char skipChar);

float version that can ignore skipChars

    int getString(char *post_string,char *buf,int length);

Puts characters into the given buffer until the post_string is detected.  The end of string is determined by a match of a character to the first char post_string.
String longer than the buffer length are truncated to fit.
The function returns the number of characters placed in the buffer (0 means no valid data found)

    int getString(char *pre_string,char *post_string,char *buf,int length);

Finds the pre_string and then puts the following characters into the given buffer until the post_string is detected.  The end of string is determined by a match of a character to the first char post_string.
String longer than the buffer length are truncated to fit.
The function returns the number of characters placed in the buffer (0 means no valid data found)
