---
layout: single
title:  "making better use of powershell history"
categories: powershell
tags: [profile, get-history, function, "100" ]
excerpt: " Often I want to save the command I have just run for later use, but I don't want to leave the console and break my concentration. I added `save-cmd` function my $Profile to pull the last item in my history, and save it to a file in my personal powershell repo. "
comments: false
date:   2017-10-17 11:11:01
modified: 2017-10-17 22:02:45


---
# ![a screenshot]({{ site.url }}/assets/images/vscode-powershell-circle.png)
The native powershell `Get-History` cmdlet only gives items from the open console and all history is lost when you close the console. Similar to `.bash_history` in Linux, the command line property from the `Get-History` output are also written to a file called `Console_history.txt`, which is ueful as a more persistant history.

### function hist
I added this to my powershell profile to make using `Console_history.txt' more fun:
{% highlight powershell linenos %}
$history = "~\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" 
function hist {get-content $history -tail 512}
{% endhighlight %}

the `hist` function gives me quick access to the contents of my `Console_history.txt`

### function save-cmd
Often I want to save the command I have just run for later use, but I don't want to leave the console and break my concentration. I added `save-cmd` function my $Profile to pull the last item in my history, and save it to a file in my personal powershell repo. 

{% highlight powershell linenos %}
$shist = "~\Documents\GitLab\powershell\consolehistory_saved.txt"
function shist {Get-Content $shist}
function save-cmd { (get-history | Select-Object -Last 1).commandline | out-file -FilePath $hist -Append -NoClobber -Force }
{% endhighlight %}

The command below will add the last item in your history to your `$Profile`
{% highlight powershell linenos %}
(get-history | Select-Object -Last 1).commandline | out-file -FilePath $profile -Append -Force -NoClobber -Encoding utf8 ; & $Profile
{% endhighlight %}


### go deeper
What will I find if I search the PSGallery for items related to "history"?
{% highlight powershell linenos %}
Find-Script *history* | Select-Object name, Author, CompanyName,description
Name                   Author         CompanyName Description
----                   ------         ----------- -----------
Invoke-SelectedHistory Jeffrey Snover jsnover     Show the command line history and Invoke the selected items
{% endhighlight %}

written by Mr. Monad Manifesto!
{% highlight powershell linenos %}
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
I should have searched PSGallery earlier. I think I'll modify this script so that instead of pulling from Get-History, it pulls from a csv which will contain my `Save-Cmd` output. I'll have to also modify `Save-Cmd` function so that it stores the object output of `get-history` instead of just the command line. Then i can make use of the gui and `invoke-command` feature in jsnover's script to work as a cool launcher for saved one-liners.
