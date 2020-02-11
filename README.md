# BTree-Indexing
 
This program implements B+ tree indexing.  It reads a text file containing data and build the index, treating the first n columns as the key, where n will be specified by the user.

The structure of the index is as follows: First 256 bytes: Name of the text file you have indexed.  This is blank-filled on the right.  I allocated 1K, since i required other meta data, and to read it whole as a block.  (The size of the key is one such piece of metadata; see 1 below.) 

The rest of the file is 1K blocks of index data, according to the way a B+ tree is structured.  This implies that the program reads in a block of data, manipulate it, and possibly (if you are inserting key/pointer pairs) write it back out as a block.I use a long (8-byte) record address for your pointers.  These "pointers" are the byte offset in the text file of the data record. Thus the first record is at offset 0, the second is at offset of 0 plus the length of the first record, and so on.  Note that the structure mixes text data (the key) and binary data (the pointer.) 

Everythin is read into memory, doing the manipulation, and writing it out. We use at most 3 disk blocks of the index file. This is because, when we’re inserting, we will need to read a node, and if we must split it, we’ll need another 1K block for half of the keys, and possibly a third one if we create a new root.

The program runs entirely from the command line. Each of the functions and their command-line parameters are given below, assuming the name of the program is INDEX:

1. <b>Create an index.</b> 
 This takes four parameters:
 
    A.	-create

    B.	The name of a text file to be indexed.

    C.	The name of an output index that is created.  Using the extension.indx for this.

    D.	How many bytes of the record, starting from the first position, are considered to be the key. If the record contains fewer characters than the key length, pad the key on the right with blanks.
    If the output file exists, overwrite it. 

Build the index by starting with a blank structure and inserting new keys. There may be duplicate keys in the input file. Your program should list them and the line number of the input file on which they occur. Duplicates are not inserted into the B+ tree.  The command line is:

> INDEX -create input_file output_file key_size
 
For example:

> INDEX -create TestData.txt MyIndex.indx 15

Creates an index with a key length of 15 bytes.  

> INDEX –create TestData.txt MyIndex.indx 21

Creates an index with a key length of 21 bytes.  

2. <b>Find a record by key. </b>

This displays the entire record, including the key, and gives its position, in bytes, within the file. If the key is not in the index, the program gives a message to that effect.  As you can guess, the reason for having the name of the text file in the B+ tree index is so we can find it for this and the subsequent command.  The command line is:

> INDEX -find index_filename key

for example, if you ran this command line:

> INDEX -find MyIndex.indx 11111111111111A

the program displays a message that the key was not found.  The command line 

> INDEX -find MyIndex.indx  64541668700164B

would display:  At 2127, record:  64541668700164B ANESTH, BIOPSY OF NOSE

(This presumes that the byte offset of this record from the start of the file is 2127).
  
If the key you supply is longer than the key length specified, the program truncates it. If it is shorter, it is padded on the right with blanks.

3.<b> Insert a new text record. </b>

As with creating a new index, the first <n> bytes contain a unique key. We will get the number of bytes by reading the header record in the index file.  If the key is not in the index, write the record at the end of the data file, then inserting the key into the index structure with the pointer to the start of the new record.  Display a message indicating success and the position of the new record in the text file.  If the key is already in the index, display a message to that effect and do not insert the record.  The program inserts the key into the index and the record into the text file. The record needs to be in quotes because it could contain spaces and other punctuation. This has the following command line:
 
 > INDEX -insert index_filename "new text line to be inserted." 

For example, the following command line would attempt to insert a record into the file.  From the previous example, we know its key is already in your index, so you would get an error message:

 > INDEX -insert MyIndex.indx “64541668700164B Some new Record"

This command line would return success:

 > INDEX -insert MyIndex.indx "11111111111111D Some new Record"

4. List sequential records. 

The program shows the record containing the given key and the next n records following it, if there are that many. If no record exists with the given key, show records that contain the next-larger key and give a message that the actual key was not found. As with the -find function, shows the records in order by key and their position within the text file. This traverses the tree in key order, which are all the connected leaf nodes in the B+ tree.  Here is the command line:

 > INDEX -list index_filename starting_key count
