## Shell history
### Search shell history
*CTRL+R* starts an interactive search through the shell history.

*CTRL+R* again cycles through the search results

*CTRL-G* ends the search

```history | grep <keyword>``` uses grep for the search.

### Delete shell history
If you write something in the shell that should not be there, passwords in the clear f.eks. it is possible to delete it from the shell history.  
```history -c``` clears the history of the current shell. It will remove all the history of the current shell session that has not been written to the system shell history yet.  
```history -w``` will overwrite he system history with the current shell history. If the shell history is empty, the new system shell history will become empty.  
```history``` will list the entire history as a number indexed list.  
```history -d <n>``` will delete the entry corresponding to the number written.  
*~/.bash_history* contains the system history, and can be edited with a text editor.