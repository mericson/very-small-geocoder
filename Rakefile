require 'net/http'
require 'json'

task :clean => [ :clean_lookups, :clean_data ] 
task :all => [ :download_data, :build_places, :build_zips ]

directory "data"
directory "public"
directory "public/lookups"

desc "Removes lookup json files"
task :clean_lookups do 
  puts "Cleaning lookups"
  `rm public/lookups/*`
end

desc "Removes original data files"
task :clean_data do 
  puts "Cleaning data"
  `rm data/*`
end


desc "Download places and zip data form geonames.org"
task :download_data => [ 'data' ] do 
  
  places = 'http://download.geonames.org/export/dump/US.zip'
  zips = 'http://download.geonames.org/export/zip/US.zip'

  puts "Downloading places.zip"
  get_store places, "data/places.zip"
  
  `unzip data/places.zip -d data -x readme.txt`
  `mv data/US.txt data/us_places.txt`
  `rm data/places.zip` 

  puts "Downloading zips.zip"
  get_store zips, "data/zips.zip"

  `unzip data/zips.zip -d data -x readme.txt`
  `mv data/US.txt data/us_zips.txt`
  `rm data/zips.zip` 

end  

desc "Creates json files from places"
task :build_places => [ 'public', 'public/lookups' ] do 

  places      = []
  coded       = {}
  
  # Read data in
  
  puts "Starting to process us_places.txt"
  File.open( 'data/us_places.txt' ) do |f|

    f.each_with_index do |l,i|
      l.chomp!
      
      place = {}
      
      d = l.split( "\t" )
      d.each_with_index do |field,i|
        place[ $HEADERS[i] ] = field 
      end
      
      if place[:population].to_i > 0  && place[:feature_class] == 'P'
        
        place[:asciiname].gsub! /^'/, '' 

        place[:two_letter_code] = place[:asciiname][0,2].downcase
        place[:class] = Math.log10( place[:population].to_i ).ceil
        places << place
      end      
      
      if i % 1000 == 0
        puts "Processed #{i} records"
      end
      
    end
  end

  # Sort by class
  puts "Sorting"
  places = places.sort do |a,b|
    if a[:class] == b[:class]
      a[:asciiname] <=> b[:asciiname]
    else
      b[:class] <=> a[:class]
    end
  end

  puts "Dividing up into two-letter codes"
  places.each do |p|
    if ! coded[ p[:two_letter_code] ] 
      coded[ p[:two_letter_code] ] = []
    end
    coded[ p[:two_letter_code] ] << p
  end

  puts "Outputing all the files"
  
  ('aa'..'zz').each do |k|

    File.open( "public/lookups/#{k}.json", "w" ) do |f|

      suggestions = []
      if coded[k]
        coded[k].each do |p|
          suggestions << { value: "#{p[:asciiname]}, #{$ABBRS[p[:admin1_code]]}" , data: { lat: p[:latitude], lon: p[:longitude]  } }
        end
      end
      
      json = { 
        suggestions: suggestions,
      }

      f.puts json.to_json
      puts "Generated #{suggestions.length} suggestions for #{k}"
    end
    
  end
end

desc "Creates zip code files from src"
task :build_zips => [ 'public', 'public/lookups' ] do 

  places      = []
  coded       = {}
  
  # Read data in
  
  puts "Starting to process us_zips.txt"
  File.open( 'data/us_zips.txt' ) do |f|

    f.each_with_index do |l,i|
      l.chomp!
      
      place = {}
      
      d = l.split( "\t" )
      d.each_with_index do |field,i|
        place[ $ZIP_HEADERS[i] ] = field 
      end

      place[:two_letter_code] = place[:postal_code][0,2].downcase
      places << place

      if i % 1000 == 0
        puts "Processed #{i} records"
      end
    
    end
  end

  # Sort by class
  puts "Sorting"
  places = places.sort do |a,b|
    a[:postal_code] <=> b[:postal_code]
  end

  puts "Dividing up into two-letter codes"
  places.each do |p|

    
    if ! coded[ p[:two_letter_code] ] 
      coded[ p[:two_letter_code] ] = []
    end

    coded[ p[:two_letter_code] ] << p
  end

  puts "Outputing all the files"
  ('00'..'99').each do |k|

    File.open( "public/lookups/#{k}.json", "w" ) do |f|
      
      suggestions = []
      
      if coded[k]
        coded[k].each do |p|
          suggestions << { value: "#{p[:postal_code]} <span class='zip-desc'>#{p[:place_name]}, #{$ABBRS[p[:admin1_code]]}</span>" , data: { lat: p[:latitude], lon: p[:longitude]  } }
        end
      end
      
      json = { 
        suggestions: suggestions,
      }

      f.puts json.to_json
      puts "Generated #{suggestions.length} suggestions for #{k}"
    end
    
  end


end


$HEADERS = [ :geonameid, :name, :asciiname, :alternatenames, :latitude, 
             :longitude, :feature_class, :feature_code, :country_code, :cc2, 
             :admin1_code, :admin2_code, :admin3_code, :admin4_code, :population, 
             :elevation, :dem, :timezone, :modification_date ] 

$ZIP_HEADERS = [ :country_code, :postal_code, :place_name, 
                 :admin1_name, :admin1_code, 
                 :admin2_name, :admin2_code,
                 :admin3_name, :admin3_code,
                 :latitude, :longitude, :accuracy ]


$ABBRS = { 'AK' => 'Alaska', 'AL' => 'Ala.', 'AR' => 'Ark.', 'AZ' => 'Ariz.', 'CA' => 'Calif.', 'CO' => 'Colo.', 'CT' => 'Conn.', 'DC' => 'D.C.', 'DE' => 'Del.', 'FL' => 'Fla.', 'GA' => 'Ga.', 'HI' => 'Hawaii', 'IA' => 'Iowa', 'ID' => 'Idaho', 'IL' => 'Ill.', 'IN' => 'Ind.', 'KS' => 'Kan.', 'KY' => 'Ky.', 'LA' => 'La.', 'MA' => 'Mass.', 'MD' => 'Md.', 'ME' => 'Maine', 'MI' => 'Mich.', 'MN' => 'Minn.', 'MO' => 'Mo.', 'MS' => 'Miss.', 'MT' => 'Mont.', 'NC' => 'N.C.', 'ND' => 'N.D.', 'NE' => 'Neb.', 'NH' => 'N.H.', 'NJ' => 'N.J.', 'NM' => 'N.M.', 'NV' => 'Nev.', 'NY' => 'N.Y.', 'OH' => 'Ohio', 'OK' => 'Okla.', 'OR' => 'Ore.', 'PA' => 'Pa.', 'PR' => 'P.R.', 'RI' => 'R.I.', 'SC' => 'S.C.', 'SD' => 'S.D.', 'TN' => 'Tenn.', 'TX' => 'Tex.', 'UT' => 'Utah', 'VA' => 'Va.', 'VT' => 'Vt.', 'WA' => 'Wash.', 'WI' => 'Wis.', 'WV' => 'W.V.', 'WY' => 'Wyo.' }


def get_store( url, dest )
  

  File.open(dest,'w'){ |f|
    uri = URI.parse(url)
    Net::HTTP.start(uri.host,uri.port){ |http| 
      http.request_get(uri.path){ |res| 
        res.read_body{ |seg|
          f << seg
          sleep 0.005 
        }
      }
    }
  }


end 
