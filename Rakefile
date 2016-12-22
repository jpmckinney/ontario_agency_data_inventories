require 'csv'
require 'json'
require 'set'

require 'open-uri/cached'
require 'open_uri_redirections'
require 'nokogiri'

task :agencies do
  url = 'https://api.ontario.ca/api/drupal/page%2Fagency-accountability?fields=body'
  Nokogiri::HTML(JSON.load(open(url).read)['body']['und'][0]['value']).xpath('//h3/following-sibling::p//a').each do |a|
    url = a['href'].strip
    if url.start_with?('/')
      # TODO
    else
      puts "#{a.text}\t#{url}"
    end
  end
end

task :inventories do
  # https://docs.google.com/spreadsheets/d/1kBSjnw-HeZn7qUs0NRIdrnhhvDilWZ7kvnglJ9p4XDQ/edit#gid=0
  url = 'https://docs.google.com/spreadsheets/d/1kBSjnw-HeZn7qUs0NRIdrnhhvDilWZ7kvnglJ9p4XDQ/pub?gid=0&single=true&output=CSV'
  CSV.parse(open(url).read, headers: true) do |row|
    url = row.fetch('URL')
    if url
      if row.fetch('Format') == 'HTML'
        inventory_url = url
      else
        begin
          # Avoid security, compression and redirection errors.
          data = open(url.sub(/\Ahttps\b/, 'http'), 'Accept-Encoding' => 'plain', allow_redirections: :safe).read

          document = Nokogiri::HTML(data)

          xpaths = %w(csv xls).flat_map do |extension|
            xpath = %(//@href[contains(., ".#{extension}")][contains(., "ventory")])
            [%(#{xpath}[not(contains(., "FR"))]), xpath]
          end + [
            '//@href[contains(., ".csv")][contains(., "OpenData")]',
            '//@href[contains(., ".csv")][contains(., "data_set")]',
            '//@href[contains(., ".pdf")][contains(., "nventory")][contains(., "ata")][not(contains(., "Fr"))]',
            '//a[contains(.//text(), "ventory")][not(contains(.//text(), "Fr"))]',
            '//a[contains(.//text(), "Open Data")]',
          ]

          hrefs = xpaths.each do |xpath|
            matches = document.xpath(xpath.to_s).map do |node|
              if node.respond_to?(:value)
                node.value
              else
                node['href']
              end
            end.uniq.reject do |href|
              [url, '#', 'https://www.ontario.ca/page/ontarios-open-data-directive'].include?(href)
            end

            if matches.any?
              break matches
            end
          end

          # If no inventory URL is found.
          if hrefs == xpaths
            # Don't match text in script tags.
            document.xpath('//script').remove

            # If there is a reason it was not found:
            if row['Reason']
              # Check if the reason is still correct.
              if row['Reason']['No mention']
                words = row['Reason'].scan(/"([^"]+)"/).flatten
                if document.text[Regexp.new(words.join('|'), Regexp::IGNORECASE)]
                  $stderr.puts "#{url}: expected 'Reason' to not include #{words.join(' or ')}"
                end
              end
            else
              if document.text[/data|inventory/i]
                $stderr.puts "#{url}: expected 'Reason' to not be nil"
              else
                $stderr.puts %(#{url}: expected 'Reason' to be 'No mention of "data" or "inventory"')
              end
            end
          else
            if row['Reason']
              $stderr.puts "#{url}: expected 'Reason' to be nil"
            end

            inventory_urls = hrefs.map do |href|
              parsed = URI.parse(url)
              if href.start_with?('/')
                "#{parsed.scheme}://#{parsed.host}#{href}"
              elsif !href.start_with?('http')
                "#{url}#{href}"
              else
                href
              end
            end

            puts CSV.generate_line([row['Name'], *inventory_urls])
          end
        rescue OpenURI::HTTPError => e
          $stderr.puts "#{url}: #{e}"
        end
      end
    end
  end
end