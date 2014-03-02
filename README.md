PutOnAHelmet
============

Set of build script/tasks that you can use to easily run to install git pre-commit hooks for such things as running tests before your code is committed.


### PowerShell [PSake](https://github.com/psake/psake)
```powershell

task putOnAHelmet -Description "Places a git pre-commit hook that runs the 'test' psake command before each commit. You can skip the pre-commit by typing "git commit -n ..." {
	$cmd = '#!/bin/sh
#***********
exec powershell -NoProfile -command "&{ import-module C:\Chocolatey\lib\psake.4.2.0.1\tools\psake.psm1; Invoke-Psake Test; exit !(\$psake.build_success); }"
#***********
'
	$hookPath = ".git/hooks/pre-commit"
	if( test-path $hookPath ){
		throw "The git pre-commit hook file already exists. Verify that this command is in there.$cmd";
	} else {
		cp "$hookPath.sample" $hookPath
		$hookFile = cat $hookPath | Out-String
		$hookFile = $hookFile.Replace('exec git diff-index --check --cached $against --', '#exec git diff-index --check --cached $against --')
		$hookFile = $hookFile + $cmd
		sc $hookPath $hookFile
	}
}
```


### Ruby [Rake](http://rake.rubyforge.org/)
```ruby

desc 'Places a git pre-commit hook that runs the "rake test" command before each commit. You can skip the pre-commit by typing "git commit -n ...'
task :putOnAHelmet do
    fileContents = <<-eos
          !bin/sh
          rake test
        eos
    File.open('.git/hooks/pre-commit', 'w') {|file|
        file.write(fileContents)
    }
    sh "chmod +x .git/hooks/pre-commit"

    # Done!
    puts 'pre-commit hook installed'
end

```
