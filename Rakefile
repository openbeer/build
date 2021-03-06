#######################################
# build script (Ruby make)
#
#  use:
#   $ rake   or
#   $ rake build     - to build beer.db from scratch
#
#   $ rake update    - to update beer.db
#
#   $ rake -T        - show all tasks


# stdlibs
require 'yaml'
require 'erb'
require 'pp'

# 3rd party libs/gems
require 'worlddb'         # check/todo: require worlddb/models - why, why not??
require 'beerdb'          # check/todo: require beerdb/models - why, why not??
require 'logutils/activerecord'



BUILD_DIR = "./build"

## our code
require './settings'

## load database config from external file (easier to configure/change)
DB_HASH = YAML.load( ERB.new( File.read( './database.yml' )).result )
DB_CONFIG = DB_HASH[ 'default' ] ## for now just always use default section/entr




task :default => :build

directory BUILD_DIR


task :clean do
  ## note: was sqlite3 only (specific) => rm BEER_DB_PATH if File.exists?( BEER_DB_PATH )

  db_adapter = DB_CONFIG[ 'adapter' ]
  ### for sqlite3 delete/remove single-file database
  if db_adapter == 'sqlite3'
    db_database = DB_CONFIG[ 'database' ]
    rm db_database if File.exists?( db_database )
  else
    puts " clean: do nothing; no clean steps configured for db adapter >#{db_adapter}<"
  end
end


task :env => BUILD_DIR do
  pp DB_CONFIG
  ActiveRecord::Base.establish_connection( DB_CONFIG )
end


task :config  => :env do
  logger = LogUtils::Logger.root
  # logger.level = :info
   ## log all warns, errors, fatals to db
  LogDb.setup
  logger.warn "Rakefile - #{Time.now}" # say hello; log to db (warn level min)
end

task :configworld => :config do
  logger = LogUtils::Logger.root
  logger.level = :info
end

task :configbeer => :config do
  logger = LogUtils::Logger.root

  ## try first
  ### use DEBUG=t or DEBUG=f
  ### or alternative LOG|LOGLEVEL=debug|info|warn|error

  debug_key = ENV['DEBUG']
  if debug_key.nil?
    ## try log_key as "fallback"
    ## - env variable that lets you configure log level
    log_key = ENV['LOG'] || ENV['LOGLEVEL'] || 'debug'
    puts " using LOGLEVEL >#{log_key}<"
    logger.level = log_key.to_sym
  else
    if ['true', 't', 'yes', 'y'].include?( debug_key.downcase )
      logger.level = :debug
    else
      logger.level = :info
    end
  end
end



task :create => :env do
  BeerDb.create_all
end

task :importworld => :configworld do

  from_zip = false

  if from_zip
    WorldDb.read_setup_from_zip( 'world.db-master', 'setups/sport.db.admin', './build', skip_tags: true )
    WorldDb.read_setup_from_zip( 'austria.db-master', 'setups/all', './build' )
  else
    # populate world tables
    WorldDb.read_setup( 'setups/sport.db.admin', WORLD_DB_INCLUDE_PATH, skip_tags: true )
    WorldDb.read_setup( 'setups/all', AUSTRIA_DB_INCLUDE_PATH )
  end
end


## for testing loading/importing from zip
task :importat => :configworld do
  TagDb.delete!    ## NOTE: also remove taggings/tags
  WorldDb.read_setup_from_zip( 'austria.db-master', 'setups/all', './build' )
end



task :importbuiltin => :env do
  # BeerDb.read_builtin
end


############################################
# add more tasks (keep build script modular)
Dir.glob('./tasks/**/*.rake').each do |r|
puts " importing task >#{r}<..."
import r
# see blog.smartlogicsolutions.com/2009/05/26/including-external-rake-files-in-your-projects-rakefile-keep-your-rake-tasks-organized/
end



#########################################################
# note: change deps to what you want to import for now

##
# default to at (if no key given)
#
# e.g. use like
# $ rake update DATA=at or
# $ rake build DATA=at
# etc.

DATA_KEY = ENV['DATA'] || ENV['DATASET'] || ENV['FX'] || ENV['FIXTURES'] || 'at'
puts " using DATA_KEY >#{DATA_KEY}<"

task :importbeer => [:configbeer, DATA_KEY.to_sym] do
  # nothing here
end


task :deletebeer => :env do
  TagDb.delete!    ## NOTE: also remove taggings/tags
  BeerDb.delete!
end


desc 'build beer.db from scratch (default)'
task :build => [:clean, :create, :importworld, :importbeer] do
  puts 'Done.'
end

desc 'update beer.db'
task :update => [:deletebeer, :importbeer] do
  puts 'Done.'
end


# desc 'pull (auto-update) beer.db from upstream sources'
# task :pull => :env do
#  BeerDb.update!
#  puts 'Done.'
# end

