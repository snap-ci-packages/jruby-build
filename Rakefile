require 'rubygems'
require 'bundler/setup'

require 'rake/clean'

distro = nil
fpm_opts = ""

if File.exist?('/etc/system-release') && File.read('/etc/redhat-release') =~ /centos|redhat|fedora|amazon/i
  distro = 'rpm'
  fpm_opts << " --rpm-user root --rpm-group root "
elsif File.exist?('/etc/os-release') && File.read('/etc/os-release') =~ /ubuntu|debian/i
  distro = 'deb'
  fpm_opts << " --deb-user root --deb-group root "
end

unless distro
  $stderr.puts "Don't know what distro I'm running on -- not sure if I can build!"
end

CLEAN.include("downloads")
CLEAN.include("jailed-root")
CLEAN.include("log")
CLEAN.include("pkg")

%w(1.6.6 1.6.7.2 1.6.7 1.6.8 1.7.0 1.7.1 1.7.2 1.7.3 1.7.4 1.7.5 1.7.6).each do |version|
  namespace version do
    prefix = File.join("/opt/local/jruby", version)

    task :init do
      mkdir_p "log"
      mkdir_p "pkg"
      mkdir_p "downloads"
      mkdir_p "jailed-root"
    end

    task :download do
      cd 'downloads' do
        url, checksum = %x[curl --fail https://raw.github.com/sstephenson/ruby-build/master/share/ruby-build/jruby-#{version} 2>/dev/null].lines.grep(/amazonaws/).first.gsub('"', '').split[2].split('#')
        jruby_source = File.basename(url)
        sh("curl --fail #{url} > #{jruby_source} 2>/dev/null")
        sh("echo '#{checksum}  #{jruby_source}' > #{jruby_source}.md5")
        sh("md5sum --check --status #{jruby_source}.md5")
      end
    end

    task :unpack do
      rm_rf "jailed-root"
      jruby_jailed_root = File.join("jailed-root", prefix)
      mkdir_p jruby_jailed_root
      sh("tar -zxf downloads/jruby-bin-#{version}.tar.gz -C #{jruby_jailed_root} --strip-components=1")
    end

    task :fpm do
      mkdir_p "pkg"
      description_string = %Q{JRuby is a 100% Java implementation of the Ruby programming language. It is Ruby for the JVM. JRuby provides a complete set of core "builtin" classes and syntax for the Ruby language, as well as most of the Ruby Standard Libraries.}
      release = ENV['GO_PIPELINE_COUNTER'] || ENV['RELEASE'] || 1
      cd "pkg" do
        sh(%Q{
          bundle exec fpm -s dir -t #{distro} --name jruby-#{version} -a x86_64 --version "#{version}" -C ../jailed-root --directories #{prefix} --verbose #{fpm_opts} --maintainer snap-ci@thoughtworks.com --vendor snap-ci@thoughtworks.com --url http://snap-ci.com --description "#{description_string}" --iteration #{release} --license '(CPL or GPLv2+ or LGPLv2+) and BSD and (GPLv2 or Ruby) and (BSD or Ruby)' .
        })
      end
    end

    desc "build and package ruby-#{version}"
    task :all => [:clean, :init, :download, :unpack, :fpm]
  end

  task :default => "#{version}:all"
end

desc "build all rubies"
task :default
