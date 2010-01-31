require 'pp'

class Find
	def initialize(block)
		@include = []
		@exclude = []
		
		instance_eval &block
	end
	
	def include(files)
		if files.is_a? Array
			@include += files
		else
			@include[@include.size] = files
		end
	end
	
	def exclude(files)
		@exclude += files
	end
	
	def find
		files = []
		@include.each do |path|
			if path =~ /\.\.\.\//
				(0...10).each do |i|
					files += Dir.glob(path.sub '.../', ('*/'*i))
				end
			else
				files += [path]
			end
		end
		
		files.map! do |file|
			if @exclude.map do |exclude|
						if file[0...exclude.size] == exclude then true
						else false
						end
					end.include? true
				nil
			else file
			end
		end
		files.compact
	end
end

def find(&block)
	Find.new(block).find
end

def boo(out, flags=[], files=[], &block)
	if block != nil
		files += find &block
	end
	
	target = 
		if out =~ /\.dll$/ then 'library'
		elsif out =~ /\.win\.exe$/ then 'winexe'
		else 'exe'
		end
	
	references = files.map do |file|
			if file =~ /\.dll$/ then file
			else nil
			end
		end.compact
	
	files = files.map do |file|
			if references.include? file then nil
			else file
			end
		end.compact
	
	references.map! { |file| "-reference:#{file}" }
	
	file out => files do
		sh 'booc', "-o:#{out}", "-target:#{target}", *references, *flags, *files
	end
	Rake::Task[out].invoke
end

task :default => [:cilly, :test]

task :obj do
	if not FileTest::directory? 'Obj'
		Dir::mkdir 'Obj'
	end
end

task :cilly => [:obj] do
	boo 'Obj/Cilly.dll' do
		include 'Mono.Cecil.dll'
		include 'Cilly/.../*.boo'
	end
	
	boo 'Obj/Cilly.exe' do
		include 'Obj/Cilly.dll'
		include 'CillyApp/.../*.boo'
	end
	
	boo 'Obj/Cilly.CilReader.dll' do
		include 'Obj/Cilly.dll'
		include 'Elements/CilReader/*.boo'
	end
end

task :test => [:cilly] do
end
