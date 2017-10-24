---
layout: single
title:  "making better use of powershell history"
categories: powershell
tags: [$profile, function, "100" ]
excerpt: " Often I want to save the command I have just run for later use, but I don't want to leave the console and break my concentration. I added `save-cmd` function my $Profile to pull the last item in my history, and save it to a file in my personal powershell repo. "
comments: true
date:   2017-10-17 11:11:01
modified: 2017-10-20 13:41:58


---
# ![a screenshot]({{ site.url }}/assets/images/vscode-powershell-circle.png)
The native powershell `Get-History` cmdlet returns results from the open console only and this history is lost when you close that console. But similarly to `.bash_history` in Linux, the command line property from the `Get-History` output are also written to a file called `Console_history.txt`, which is useful as a more persistent history.

### function hist
I added this to my powershell profile to make using `Console_history.txt` more fun:
{% highlight powershell %}
$history = "~\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" 
function hist {get-content $history -tail 512}
{% endhighlight %}
<br>
the `hist` function gives me quick access to the contents of my `Console_history.txt`

### function save-cmd
Often I want to save the command I have just run for later use, but I don't want to leave the console and break my concentration. I added `save-cmd` function my $Profile to pull the last item in my history, and save it to a file in my personal powershell repo. I can retrieve the content of this file with the `shist` function.

{% highlight powershell %}
$shist = "~\Documents\GitLab\powershell\consolehistory_saved.txt"
function shist {Get-Content $shist}
function save-cmd { (get-history | Select-Object -Last 1).commandline | out-file -FilePath $hist -Append -NoClobber -Force }
{% endhighlight %}
<br>
The command below will add the last item in your history to your `$Profile`
{% highlight powershell %}
(get-history | Select-Object -Last 1).commandline | out-file -FilePath $profile -Append -Force -NoClobber -Encoding utf8 ; & $Profile
{% endhighlight %}


### go deeper
What will I find if I search the PSGallery for items related to "history"?
{% highlight powershell %}
Find-Script *history* | Select-Object name, Author, CompanyName,description
Name                   Author         CompanyName Description
----                   ------         ----------- -----------
Invoke-SelectedHistory Jeffrey Snover jsnover     Show the command line history and Invoke the selected items
{% endhighlight %}
<br>
written by **Mr. Monad Manifesto**!
{% highlight powershell %}
Find-Script *history* | Save-Script -Path . -Force
cat .\Invoke-SelectedHistory.ps1

.DESCRIPTION
   Show the command line history and Invoke the selected items

.DESCRIPTION
   Essentially this does:
    PS> Get-History -count:$count | Out-Gridview -Passthru | Invoke-History

   The benefit is that you can do searching and multiple selections.
.EXAMPLE
   PS> ISH -count 200 -verbose
{% endhighlight %}
pretty neat alias in that script:  `ISH`

### lessons learned..
I should have searched PSGallery earlier. I plan to modify this script so that instead of pulling from Get-History, it pulls from a csv which will contain my `Save-Cmd` output. I'll have to also modify my `Save-Cmd` function so that it stores the object output of `get-history` instead of just the command line. Then i can make use of the gui and `invoke-command` feature in jsnover's script to work as a cool launcher for saved one-liners.
