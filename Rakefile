# CollectionBuilder-SA helper tasks

###############################################################################
# TASK: deploy
###############################################################################

desc "Build site with production env"
task :deploy do
  ENV["JEKYLL_ENV"] = "production"
  exec("bundle exec jekyll build")
end

###############################################################################
# Helper Functions
###############################################################################

$ensure_dir_exist = ->(dir) { if !Dir.exist?(dir) then Dir.mkdir(dir) end }

def prompt_user_for_confirmation message
  response = nil
  while true do
    # Use print instead of puts to avoid trailing \n.
    print "#{message} (Y/n): "
    $stdout.flush
    response =
      case STDIN.gets.chomp.downcase
      when "", "y"
        true
      when "n"
        false
      else
        nil
      end
    if response != nil
      return response
    end
    puts "Please enter \"y\" or \"n\""
  end
end

###############################################################################
# TASK: generate_derivatives
###############################################################################

desc "Generate derivative image files from collection objects"
task :generate_derivatives, [:thumbs_size, :small_size, :density, :missing, :im_executable] do |t, args|

  # set default arguments 
  args.with_defaults(
    :thumbs_size => "400x400",
    :small_size => "800x800",
    :density => "300",
    :missing => "true",
    :im_executable => "magick",
  )

  # set the folder locations
  objects_dir = "objects"
  thumb_image_dir = "objects/thumbs"
  small_image_dir = "objects/small"

  # Ensure that the output directories exist.
  [thumb_image_dir, small_image_dir].each &$ensure_dir_exist

  # support these file types
  EXTNAME_TYPE_MAP = {
    '.tiff' => :image,
    '.tif' => :image,  
    '.jpg' => :image,
    '.png' => :image,
    '.pdf' => :pdf
  }

  # Iterate over files in objects directory.
  Dir.glob(File.join([objects_dir, '*'])).each do |filename|
    # Ignore subdirectories.
    if File.directory? filename
      next
    end

    # Determine the file type and skip if unsupported.
    extname = File.extname(filename).downcase
    file_type = EXTNAME_TYPE_MAP[extname]
    if !file_type
      puts "Skipping file with unsupported extension: #{filename}"
      next
    end

    # Define the file-type-specific ImageMagick command prefix.
    cmd_prefix =
      case file_type
      when :image then "#{args.im_executable} #{filename}"
      when :pdf then "#{args.im_executable} -density #{args.density} #{filename}[0]"
      end

    # Get the lowercase filename without any leading path and extension.
    base_filename = File.basename(filename)[0..-(extname.length + 1)].downcase

    # Generate the thumb image.
    thumb_filename=File.join([thumb_image_dir, "#{base_filename}_th.jpg"])
    if args.missing == 'false' or !File.exist?(thumb_filename)
      puts "Creating: #{thumb_filename}";
      system("#{cmd_prefix} -resize #{args.thumbs_size} -flatten #{thumb_filename}")
    else
      puts "Skipping: #{thumb_filename} already exist"
    end

    # Generate the small image.
    small_filename = File.join([small_image_dir, "#{base_filename}_sm.jpg"])
    if args.missing == 'false' or !File.exist?(small_filename)
      puts "Creating: #{small_filename}";
      system("#{cmd_prefix} -resize #{args.small_size} -flatten #{small_filename}")
    else
      puts "Skipping: #{small_filename} already exist"
    end
  end
end
