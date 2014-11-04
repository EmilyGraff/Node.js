Node.js
=======

Node.js
  process.exit(1)
 })
 
-/* commander.js */
-var commands = {
-  'create': 'create [path]',
-  'build': 'build [path]',
-  'serve': 'serve [path] [options]',
-  'new': 'new <page|post> [path]',
-  'about': 'about'
-}
+var commands = [
+  {
+    cmd: 'create [path]',
+    desc: 'Create a new website at given path',
+    func: cli.create
+  },
+  {
+    cmd: 'build [path]',
+    desc: 'Compile a website to its buildDir',
+    func: cli.build
+  },
+  {
+    cmd: 'serve [path] [options]',
+    desc: 'Start a server on localhost:port and watch changes',
+    func: cli.serve
+  },
+  {
+    cmd: 'new <page|post> [path]',
+    desc: 'Create a new post or page',
+    func: cli.file
+  },
+  {
+    cmd: 'about',
+    desc: 'About Equiprose',
+    func: cli.about
+  }
+]
 
 program
   .version(pjson.version)
 
-// TODO: Loop
-program
-  .command(commands['create'])
-  .description('Create a new website at given path')
-  .action(cli.create)
-
-program
-  .command(commands['build'])
-  .description('Compile a website to its buildDir')
-  .action(cli.build)
-
-program
-  .command(commands['serve'])
-  .description('Start a server on localhost:port')
-  .action(cli.serve)
-
-program
-  .command(commands['new'])
-  .description('Create a new post or page')
-  .action(cli.file)
-
-program
-  .command(commands['about'])
-  .description('About Equiprose')
-  .action(cli.about)
+commands.forEach(function (command) {
+  program
+    .command(command.cmd)
+    .description(command.desc)
+    .action(command.func)
+})
 
 program.option('-p, --port [port]', 'Specify the port on which to run the test server')
 
 program.on('--help', function () {
-  CLI.logger.log('  Examples:')
-  CLI.logger.log('')
-  CLI.logger.log('    $ equiprose create ~/website')
-  CLI.logger.log('    $ equiprose new post ~/website')
-  CLI.logger.log('    $ equiprose serve ~/website -p 9000')
-  CLI.logger.log('')
+  cli.logger.log('  Examples:')
+  cli.logger.log('')
+  cli.logger.log('    $ equiprose create ~/website')
+  cli.logger.log('    $ equiprose new post ~/website')
+  cli.logger.log('    $ equiprose serve ~/website -p 9000')
+  cli.logger.log('')
 })
 
 program.parse(process.argv)
 @@ -63,7 +63,11 @@ if (program.rawArgs.length < 3) {
   program.help()
 }
 
-if (!commands.hasOwnProperty(process.argv[2])) {
-  CLI.logger.log('  Invalid command: ' + process.argv[2])
+var filtered = commands.filter(function (cmd) {
+  return process.argv[2] === cmd.cmd
+})
+
+if (filtered.length === 0) {
+  cli.logger.log('  Invalid command: ' + process.argv[2])
   program.help()
 }
3  lib/serve.js View
 @@ -9,14 +9,11 @@
 module.exports = function (siteConfig, port, callback) {
   var http    = require('http')
   var express = require('express')
-  var path    = require('path')
   var app     = express()
-  var parser  = require('./parsing')
 
   app.configure(function () {
     app.use(express.compress())
     app.set('port', port)
-    //app.use(express.bodyParser())
     app.use(express.logger('dev'))
     app.use(express.static(siteConfig.paths.buildDir))
   })
