require "spec/rake/spectask"
require 'rake/gempackagetask'
require 'rake/extensiontask'
require 'rake/javaextensiontask'
require 'rake/contrib/rubyforgepublisher'
require 'rake/clean'
require 'rake/rdoctask'
require "benchmark"

PKG_NAME = "bcrypt-ruby"
PKG_VERSION   = "2.1.2"
PKG_FILE_NAME = "#{PKG_NAME}-#{PKG_VERSION}"
PKG_FILES = FileList[
  '[A-Z]*',
  'lib/**/*.rb', 
  'spec/**/*.rb', 
  'ext/mri/*.c',
  'ext/mri/*.h',
  'ext/mri/*.rb',
  'ext/jruby/bcrypt_jruby/BCrypt.java'
]
CLEAN.include(
  "ext/mri/*.o",
  "ext/mri/*.bundle",
  "ext/mri/*.so",
  "ext/jruby/bcrypt_jruby/*.class"
)
CLOBBER.include(
  "ext/mri/Makefile",
  "doc/coverage",
  "pkg"
)

task :default => [:compile, :spec]

desc "Run all specs"
Spec::Rake::SpecTask.new do |t|
  t.spec_files = FileList['spec/**/*_spec.rb']
  t.spec_opts = ['--color','--backtrace','--diff']
end

desc "Run all specs, with coverage testing"
Spec::Rake::SpecTask.new(:rcov) do |t|
  t.spec_files = FileList['spec/**/*_spec.rb']
  t.spec_opts = ['--color','--backtrace','--diff']
  t.rcov = true
  t.rcov_dir = 'doc/coverage'
  t.rcov_opts = ['--exclude', 'rspec,diff-lcs,rcov,_spec,_helper']
end

desc 'Generate RDoc'
rd = Rake::RDocTask.new do |rdoc|
  rdoc.rdoc_dir = 'doc/rdoc'
  rdoc.options << '--title' << 'bcrypt-ruby' << '--line-numbers' << '--inline-source' << '--main' << 'README'
  rdoc.template = ENV['TEMPLATE'] if ENV['TEMPLATE']
  rdoc.rdoc_files.include('README', 'COPYING', 'CHANGELOG', 'lib/**/*.rb')
end

spec = Gem::Specification.new do |s|
  s.name = PKG_NAME
  s.version = PKG_VERSION
  s.summary = "OpenBSD's bcrypt() password hashing algorithm."
  s.description = <<-EOF
    bcrypt() is a sophisticated and secure hash algorithm designed by The OpenBSD project
    for hashing passwords. bcrypt-ruby provides a simple, humane wrapper for safely handling
    passwords.
  EOF

  s.files = PKG_FILES.to_a
  s.require_path = 'lib'
  s.add_development_dependency 'rake-compiler'

  s.has_rdoc = true
  s.rdoc_options = rd.options
  s.extra_rdoc_files = rd.rdoc_files.to_a
  
  s.extensions = FileList["ext/mri/extconf.rb"].to_a
  
  s.authors = ["Coda Hale"]
  s.email = "coda.hale@gmail.com"
  s.homepage = "http://bcrypt-ruby.rubyforge.org"
  s.rubyforge_project = "bcrypt-ruby"
end

Rake::GemPackageTask.new(spec) do |pkg|
  pkg.need_zip = true
  pkg.need_tar = true
end

if RUBY_PLATFORM =~ /java/
  Rake::JavaExtensionTask.new('bcrypt_ext', spec) do |ext|
    ext.ext_dir = 'ext/jruby'
  end
else
  Rake::ExtensionTask.new("bcrypt_ext", spec) do |ext|
    ext.ext_dir = 'ext/mri'
    ext.cross_compile = true
    ext.cross_platform = ['x86-mingw32', 'x86-mswin32-60']

    # inject 1.8/1.9 pure-ruby entry point
    ext.cross_compiling do |spec|
      spec.files += ["lib/#{ext.name}.rb"]
    end
  end
end

# Entry point for fat-binary gems on win32
file("lib/bcrypt_ext.rb") do |t|
  File.open(t.name, 'wb') do |f|
    f.write <<-eoruby
RUBY_VERSION =~ /(\\d+.\\d+)/
require "\#{$1}/#{File.basename(t.name, '.rb')}"
    eoruby
  end
  at_exit{ FileUtils.rm t.name if File.exists?(t.name) }
end

desc "Run a set of benchmarks on the compiled extension."
task :benchmark do
  TESTS = 100
  TEST_PWD = "this is a test"
  require File.expand_path(File.join(File.dirname(__FILE__), "lib", "bcrypt"))
  Benchmark.bmbm do |results|
    4.upto(10) do |n|
      results.report("cost #{n}:") { TESTS.times { BCrypt::Password.create(TEST_PWD, :cost => n) } }
    end
  end
end
