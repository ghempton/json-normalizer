# Install Transpiler

task :install_transpiler do
  if `which compile-modules`.empty?
    sh "npm install es6-module-transpiler -g"
  end
end

# Helpers

directory "browser/json-normalizer"

def amd_module(filename)
  out_name = filename.sub(/\.js$/, '.amd.js')
  output = "browser/#{out_name}"
  input = "lib/#{filename}"

  file output => ["browser/json-normalizer", input] do
    open output, "w" do |file|
      file.puts %x{compile-modules --type amd --anonymous -s < #{input}}
    end
  end

  output
end

def read(file)
  File.read(file)
end

def named_module(name, filename)
  name = name.match(/^(?:vendor\/)?(.*)\.js$/)[1]
  body = read(filename)
  body.sub(/define\(/, "define(#{name.inspect},")
end

modules = Dir.chdir "lib" do
  Dir["**/*.js"] - ["vendor/loader.js"]
end

# Build AMD Modules

amd_modules = modules.reduce({}) do |hash, mod|
  hash.merge mod => amd_module(mod)
end

browser_dependencies = ["browser/json-normalizer"] + amd_modules.values

# Define Browser Build

file "browser/json-normalizer.js" => browser_dependencies do
  output = []
  output << %|(function() {|
  output << read("lib/vendor/loader.js")

  amd_modules.each do |name, filename|
    output << named_module(name, filename)
  end

  output << %|window.JSONNormalizer = require('json-normalizer');|
  output << %|})();|

  open("browser/json-normalizer.js", "w") do |file|
    file.puts output.join("\n")
  end
end

task :dist => [:install_transpiler, "browser/json-normalizer.js"]

desc "compile htmlbars"
task :default => :dist
