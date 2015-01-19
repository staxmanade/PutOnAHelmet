PutOnAHelmet
============

Set of build script/tasks that you can use to easily run to install git pre-commit hooks for such things as running tests before your code is committed.

- [Shell - Make](#shell)
- [Ruby - Rake](#rubyrake)
- [Gulpjs](#gulpjs)
- [PowerShell - Invoke-Build](#powershellInvokeBuild)
- [PowerShell - PSake](#powershellPsake)

<a name="shell"/>
### Shell Make
```shell
# Installs pre commit hook
installCommitHook:
	@[ ! -f .git/hooks/pre-commit ] || { echo "pre-commit hook file already exists."; exit 1; }
	@echo "#!/bin/sh\nmake test" > .git/hooks/pre-commit;
	@chmod a+x .git/hooks/pre-commit
	@echo "pre-commit hook installed!"
```

<a name="rubyrake"/>
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

    puts 'pre-commit hook installed'
end

```

<a name="gulpjs"/>
### Gulp [Gulpjs](http://gulpjs.com/)
```JavaScript
  gulp.task('putOnAHelmet', function () {
    var testFileString = '!bin/sh\ngulp test';
    var fs = require('fs');
    var preCommitHookFile = '.git/hooks/pre-commit';
    if(fs.existsSync(preCommitHookFile)) {
      throw 'pre-commit hook file already exists. Don\'t want to override it (case you have a custom special one in there..).';
    } else {
      fs.writeFileSync(preCommitHookFile, testFileString);
      require('child_process').exec('chmod +x ' + preCommitHookFile);
    }
  });
```


<a name="powershellInvokeBuild"/>
### PowerShell [Invoke-Build](https://github.com/nightroman/Invoke-Build)

```powershell
# Synopsis: Places a git pre-commit hook that runs the 'test' psake command before each commit. You can skip the pre-commit by typing "git commit -n ..."
task putOnAHelmet {
    $cmd = '#!/bin/sh
#***********
exec powershell -NoProfile -command "Invoke-Build.ps1 Test"
#***********
'
    $hookPath = ".git/hooks/pre-commit"
    if( test-path $hookPath ){
        throw "The git pre-commit hook file already exists. Verify that this command is in there. $cmd"
    } else {
        $hookFile = cat "$hookPath.sample" | Out-String
        $hookFile = $hookFile.Replace('exec git diff-index --check --cached $against --', '#exec git diff-index --check --cached $against --')
        $hookFile = $hookFile + $cmd
        sc $hookPath $hookFile
    }
    "Pre-commit hook installed!"
}
```


<a name="powershellPsake"/>
### PowerShell [PSake](https://github.com/psake/psake)
```powershell

task putOnAHelmet -Description "Places a git pre-commit hook that runs the 'test' psake command before each commit. You can skip the pre-commit by typing `"git commit -n ...`"" {
	$cmd = '#!/bin/sh
#***********
exec powershell -NoProfile -command "&{ import-module C:\Chocolatey\lib\psake.4.2.0.1\tools\psake.psm1; Invoke-Psake Test; exit !(\$psake.build_success); }"
#***********
'
	$hookPath = ".git/hooks/pre-commit"
	if( test-path $hookPath ){
		throw "The git pre-commit hook file already exists. Verify that this command is in there.$cmd";
	} else {
		$hookFile = cat "$hookPath.sample" | Out-String
		$hookFile = $hookFile.Replace('exec git diff-index --check --cached $against --', '#exec git diff-index --check --cached $against --')
		$hookFile = $hookFile + $cmd
		sc $hookPath $hookFile
	}
	echo "Pre-commit hook installed!"
}
```
