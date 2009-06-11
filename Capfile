#
# Author: Joshua Timberman <joshua@hjksolutions.com>
#
# Copyright 2008, HJK Solutions
# Portions originally written by spiceee, 
#   http://snippets.dzone.com/posts/show/6372
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#     http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# role :app, "app-prod.example.com", "db-prod.example.com"
role :app, "app-test.example.com"

# Change to latest release or desired prior version for download link below.
ree_dl_id = "51100"
ree_version = "1.8.6-20090201"

ree_tarball = "ruby-enterprise-#{ree_version}.tar.gz"
ree_source = "http://rubyforge.org/frs/download.php/#{ree_dl_id}/#{ree_tarball}"
ree_path = "/srv/ree"
# -S old_gem="system" to use the system gem, otherwise
# specify the location of the old gem command (for upgrading REE)
case old_gem
when "system"
  set(:old_gem, "/usr/bin/gem")
else
  set(:old_gem, old_gem)
end
set(:new_gem, "#{ree_path}-#{ree_version}/bin/gem")

# Uncomment to use the local gems server, also uncomment cmd in ree_gems task.
#gem_source = "http://gems"

desc "Install Ruby Enterprise Edition"
task :ree_install, :roles => :app do
  logger.info("Installing Ruby Enterprise Edition from source")
  run("mkdir -p /tmp/ree")
  run("sh -c '(cd /tmp/ree; wget #{ree_source} 2>/dev/null)'")
  run("sh -c '(cd /tmp/ree; tar zxf #{ree_tarball})'")
  sudo("sh -c '(cd /tmp/ree/ruby-enterprise-#{ree_version}; ./installer -a #{ree_path}-#{ree_version})'")
  sudo("rm -rf /tmp/ree")
end

desc "Effectivelly switch the ree softlink to use a different ree version"
task :ree_switch, :roles => :app do
  logger.info("Switching ree to version #{ree_version}")
  sudo("ln -sf #{ree_path}-#{ree_version} #{ree_path}")
end

desc "Get a list of the installed ree versions"
task :ree_show_versions, :roles => :app do
  dirs = capture "ls -ld #{ree_path}-*"
  versions = dirs.map do |l|
    matches = l.match(/#{ree_path}-(.*)$/)
    matches ? matches[1] : nil
  end.compact.sort
  logger.info("Installed ree versions: #{versions.join(', ')}")
end

desc "Install System Gems in REE"
task :ree_gems, :roles => :app do
  oldgems = capture "#{old_gem} list"
  newgems = capture "#{new_gem} list"
  # oldgems.each block written by spiceee.
  # http://snippets.dzone.com/posts/show/6372
  oldgems.each do |line|
    matches = line.match(/([A-Z].+) \(([0-9\., ]+)\)/i)
    if matches 
    then
      gem_name = matches[1]
      versions = matches[2]
      versions.split(', ').each do |ver|
        cmd = "#{new_gem} install #{gem_name} --version #{ver} --no-ri --no-rdoc" # --source #{gem_source}"
        # rubygems-update is "installed" because REE includes RubyGems 1.3.1.
        if newgems =~ /#{gem_name} \(.*#{ver}.*\)/i || gem_name =~ /rubygems-update/
        then
          logger.info("#{gem_name} #{ver} is already installed. Skipping.")
        else
          sudo(cmd)
        end
      end
    end
  end # oldgems.each
end

desc "Install REE and system gems"
task :ree_install_all, :roles => :app do
  ree_install
  ree_gems
end

desc "Install REE, system gems and switch to latest version."
task :ree_install_all_and_switch, :roles => :app do
  ree_install_all
  ree_switch
end
