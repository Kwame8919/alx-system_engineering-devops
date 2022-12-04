![funny](https://user-images.githubusercontent.com/101393080/205517041-428ac745-d9bc-4bdc-b0c6-9940c9f166d7.gif)







# 0x19. Postmortem

## Description
Any software system will eventually fail, and that failure can come stem from a wide range of possible factors: bugs, traffic spikes, security issues, hardware failures, natural disasters, human error… Failing is normal and failing is actually a great opportunity to learn and improve. Any great Software Engineer must learn from his/her mistakes to make sure that they won’t happen again. Failing is fine, but failing twice because of the same issue is not.


Issue Summary:
Start time: 10/01/19 7:00 AM (GMT), End time: 10/01/19 8:00 AM (GMT).
The wordpress page was returning a 500 status code, so the page was down for a 100% of the users.
Root cause: typo in a wordpress settings document.
Timeline:
3/11/22 7:05 AM (GMT) - The issue was detected by several users, who contacted the customer service department.
3/11/22 7:10 AM (GMT) - The issue was escalated to the System Engineering team, and the SRE.
3/11/22 7:15 AM (GMT) - They looked at the running processes on the server using ‘ps auxf’ to see if any unwanted child process was running in the background, and keeping the server from responding.
3/11/19 7:20 AM (GMT) - After seeing the processes looked fine, the team used ‘strace’ on some process ids including the ones of apache2 (the web server hosting the wordpress page).
3/11/22 7:30 AM (GMT) - strace on one of the apache2 processes was showing an infinite loop of system calls, so they looked at the second apache2 process, that was calling the system call accept4() and hanging.
3/11/22 7:35 AM (GMT) - When using curl on the page’s IP while running strace on that second apache2 process, the team realized strace was displaying a lot of errors. One of them said that the file index.html didn’t exist, but it was a misleading clue because adding that file sin the wordpress folders didn’t seem to make it work.
3/11/22 7:40 AM (GMT) - After reading carefully all the errors returned by strace, the team saw that one of them mentioned that a file didn’t exist: the file that apache2 was trying to access seemed to be terminating in ‘.phpp’, which is not a common extension for a file.
3/11/22 7:45 AM (GMT) - When looking at the wordpress settings file, /var/www/html/wp-settings.php, line 137 was trying to require that faulty file. From then, the team just removed the extra ‘p’ at the end of the extension.
3/11/22 7:50 AM (GMT) - The team only had to restart apache2 using ‘service apache2 restart’. The page was back up like normal.
Root cause and resolution:
One typo in the wordpress settings file was found, causing apache2 to not work properly.
The issue was saved by removing that typo and restarting apache2.
Corrective and preventative measures:
Setting files should not have write permissions for anyone else than the SRE, in order to avoid injection of small typos like the one that was experienced in this incident.
TODO: 
Change permissions on /var/www/html/wp-settings.php to read-only for the team.
Read carefully all setting files to look for other typos of that type.
