Adding tags to the feed
-----------------------

The nanoc helpers provide for tags and an atom feed. It might be useful to integrate the tags into the atom feed, for use on metablogs etc. To do that, copy the `blogging.rb` from the nanoc sources into your projects `lib/` and look for the `xml.entry` loop inside the `atom_feed` definition. In front of the `# Add content` commentary is a good place to add this:

	# Add tags as described in http://edward.oconnor.cx/2007/02/representing-tags-in-atom
	if a[:tags].nil? or a[:tags].empty?
	  # Do nothing
	else
	  a[:tags].each do |tag|             
	    xml.category(
	      :scheme => @config[:base_url] +  '/tag/',
	      :term => tag.to_sym,
	      :label => tag.to_sym)
	  end
	end

Adding a PubSubHubbub reference
-------------------------------

If you want to publish your Atom feed using PubSubHubbub, you can add a link to it to the header of your atom file by adding the following lines to `blogging.rb`, in the part of the `atom_feed` definition that writes the general data. Right in front of the `# Add author information` comment might be a good place.
        

	# Include pusubhubbub link if available
	if @config[:pubsubhubbub].nil? or @config[:pubsubhubbub].empty?
	  # Do nothing
	else
	  @config[:pubsubhubbub].each do |hub|             
	    xml.link(:rel => 'hub', :href => hub)
	  end
	end

Using this, you can set one or more hubs in your _config.yaml_ by adding 

	pubsubhubbub: [hub1.com, hub2.net]
