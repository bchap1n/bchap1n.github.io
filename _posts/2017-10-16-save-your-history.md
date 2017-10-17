---
layout: single
title:  "making better use of powershell history"
categories: powershell
tags: [profile, get-history, function ]
excerpt: " function save-cmd { (get-history | Select-Object -Last 1).commandline | out-file -FilePath $scmd -Append -NoClobber -Force } "
comments: false
date:   2017-10-17 11:11:01 -0800
modified: 2015-10-17


---
# ![a screenshot]({{ site.url }}/assets/images/vscode-powershell-circle.png)
The native `Get-History` cmdlet only gives items from the open console and all is lost when you close the console. Similar to `.bash_history` these items are also written to a file called `Console_hisotry.txt`, but you lose the `id` information.

### function hist
Here are a few elements I have added to my powershell profile to make using my console history more fun:

{% highlight powershell linenos %}
$history = "~\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" 
function hist {get-content $history -tail 512}
{% endhighlight %}

the `hist` function gives me quick access to the contents of my `Console_hisotry.txt` and I use it a lot!

### function save-cmd
often I find something that is worth saving but I don't want to break my concentration. I added `save-cmd` to pull the last item in my history and save it so a file in my personal powershell repo. 
{% highlight powershell linenos %}
$scmd = "~\Documents\GitLab\powershell\consolehistory_saved.txt"
function scmd {Get-Content $scmd}
function save-cmd { (get-history | Select-Object -Last 1).commandline | out-file -FilePath $scmd -Append -NoClobber -Force }
{% endhighlight %}

for fun you can use the below to add the last item in your history to your `$Profile`

{% highlight powershell linenos %}
(get-history | Select-Object -Last 1).commandline | out-file -FilePath $profile -Append -Force -NoClobber -Encoding utf8 ; & $Profile
{% endhighlight %}




### go deeper
{% highlight powershell linenos %}
Find-Script *history* | Select-Object name, Author, CompanyName,description

Name                   Author         CompanyName Description
----                   ------         ----------- -----------
Invoke-SelectedHistory Jeffrey Snover jsnover     Show the command line history and Invoke the selected items
{% endhighlight %}
whoah... written by the creator of the Monad Manifesto!


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
this may be my favorite alias of all time!:  `ISH`

### lessons learned..
I should have searched PSGallery earlier. I think I'll modify this script so that instead of pulling from Get-History, it pulls from a csv which will contain my `Save-Cmd` output. I'll have to also modify `Save-Cmd` so that it stores the object output of `get-history` instead of just the command line. Then i can make use of the gui and `invoke-command` feature in jsnover's script to work as a cool launcher for saved one-liners.
