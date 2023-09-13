# jrkelly-helpcog
This is a help cog that i use for my bot called Jr. Kelly you can just copy and paste it if you want it would be appreciated if you mention my name tho :D

```py
from typing import Any, List, Optional
import discord
from discord.components import SelectOption
from discord.ext import commands
import asyncio
from discord.interactions import Interaction


from discord.utils import MISSING

color = 0x8F00FF
            

def get_row(row, data):
    return row % len(data)




class help_cate(discord.ui.View):
    def __init__(self, data, bot):
        super().__init__()
        self.data = data
        self.bot = bot
        self.row = 0


    @discord.ui.button(label="<", style=discord.ButtonStyle.blurple)
    async def pre(self, interaction:discord.Interaction, button: discord.Button):
        await interaction.response.defer()
        data = self.data
        self.row -= 1
        ea = discord.Embed()
        frow = await get_row(self.row, self.data)
        owo = []
        

        for cmd in data[frow]:
            cmd = self.bot.get_command(cmd)


            aliases = '|'.join(cmd.aliases) if cmd.aliases else ''
            if len(aliases)>0:
                aliases = cmd.name + "|" + aliases

            else:
                aliases = cmd.name


            usage = f"```jr {aliases} {cmd.signature}```"
            des = cmd.description
            a = f"{des} \n{usage}"
            owo.append(a)

            
        ea.description = '\n'.join(owo)
        ea.set_footer(text=  "Example: jr ping|stats -> jr ping or jr stats and not jr ping|stats")
        for e in self.children:
            if e.custom_id == "pages":
                e.label = f"{frow+1}/{len(self.data)}"
        await interaction.edit_original_response(embed=ea, view=self)

 
    @discord.ui.button(label="None", disabled=True, custom_id="pages")
    async def Disabeld(self, inte, but):
        ...
    
    
    
    @discord.ui.button(label=">", style=discord.ButtonStyle.secondary)
    async def next_hep(self, interaction:discord.Interaction, button: discord.Button):
        await interaction.response.defer()
        data = self.data
        self.row += 1
        ea = discord.Embed()
        frow = await get_row(self.row, self.data)
        owo = []
        

        for cmd in data[frow]:
            cmd = self.bot.get_command(cmd)


            aliases = '|'.join(cmd.aliases) if cmd.aliases else ''
            if len(aliases)>0:
                aliases = cmd.name + "|" + aliases

            else:
                aliases = cmd.name


            usage = f"```jr {aliases} {cmd.signature}```"
            des = cmd.description
            a = f"{des} \n{usage}"
            owo.append(a)

            
        ea.description = '\n'.join(owo)
        ea.set_footer(text=  "Example: jr ping|stats -> jr ping or jr stats and not jr ping|stats")
        for e in self.children:
            if e.custom_id == "pages":
                e.label = f"{frow+1}/{len(self.data)}"
        await interaction.edit_original_response(embed=ea, view=self)





class helpselector(discord.ui.Select):
    def __init__(self, helpdata, bot:discord.Client) -> None:
        super().__init__(placeholder="Select category you want the help command about",
                        min_values=1,
                        max_values=1,
                        options=[discord.SelectOption(label="Safe Commands", description="Safe commands are safe to use anywhere anytime!", value="safe"),
                        discord.SelectOption(label="Admin Commands", description="Admin commands are safe to use anywhere anytime!", value="admin"),
                        discord.SelectOption(label="Nsfw Commands", description="Nsfw commands are safe to use anywhere anytime!", value="nsfw"),
                        discord.SelectOption(label="Afk Commands", description="Afk commands are safe to use anywhere anytime!", value="afk"),
                        discord.SelectOption(label="Search Commands", description="Explore/Search commands are safe to use anywhere anytime!", value="explore"),
                        discord.SelectOption(label="Level Commands", description="Level commands are safe to use anywhere anytime!", value="levelingsystem"),
                        discord.SelectOption(label="Economy Commands", description="Economy commands are safe to use anywhere anytime!", value="economy"),
                        discord.SelectOption(label="Pets Commands", description="Pets commands are safe to use anywhere anytime!", value="pets"),
                        discord.SelectOption(label="Vote Commands", description="Vote commands are safe to use anywhere anytime!", value="vote"),
                        discord.SelectOption(label="Welcome Commands", description="Welcome/AutoNsfw-Setup commands are safe to use anywhere anytime!", value="welcome")
                        ])
        self.data= helpdata
        self.bot = bot
        
    
    async def callback(self, interaction: discord.Interaction) -> Any:
        m = interaction.data['values'][0]
        e = discord.Embed()
        e.description = ''
        owo = []
        data = self.data[m]


        for cmd in data[0]:
            cmd = self.bot.get_command(cmd)


            aliases = '|'.join(cmd.aliases) if cmd.aliases else ''


            if len(aliases)>0:
                aliases = cmd.name + "|" + aliases

            else:
                aliases = cmd.name


            usage = f"```jr {aliases} {cmd.signature}```"
            des = cmd.description
            a = f"{des} \n{usage}"
            owo.append(a)

            
        e.description = '\n'.join(owo)
        e.set_footer(text=  "Example: jr ping|stats -> jr ping or jr stats and not jr ping|stats")

        await interaction.response.send_message(embed=e, ephemeral=True, view=help_cate(data, self.bot))








class Buttonwithselect(discord.ui.View):
    def __init__(self, pages, bot):
        super().__init__()
        self.bot = bot
        self.add_item(helpselector(pages, self.bot))








class Help(commands.Cog, name='Help'):
    def __init__(self, bot):
        self.bot = bot
        self.per_page = 3


        # for nobies
        self.pages = {}
        self.categories = {}


        # owners commands list

        self.o_pages= {}
        self.o_cat = {}
    
    async def cog_load(self) -> None:
        for command in self.bot.commands:
            
            category = command.cog_name.lower() if command.cog_name else "uncategorized"
            if command.hidden == False:
                    

                if category not in self.categories:
                    self.categories[category] = []
                self.categories[category].append(command.name)


        for category, commands_list in self.categories.items():
            self.pages[category] = [commands_list[i:i + self.per_page] for i in range(0, len(commands_list), self.per_page)]


#       # for owners\
        for command in self.bot.commands:
            category = command.cog_name or "Uncategorized"

            if category not in self.o_cat:
                self.o_cat[category] = []
            self.o_cat[category].append(command.name)


        for category, commands_list in self.categories.items():
            self.o_pages[category] = [commands_list[i:i + self.per_page] for i in range(0, len(commands_list), self.per_page)]



    @commands.command()
    @commands.guild_only()
    async def help(self, ctx, *, command=None):
        if not command:


            e = discord.Embed()
            data = await self.bot.collection_main.find_one({"user_id": str(ctx.author.id)})
            if data:
                ti = "You are a verified user!"
            else: 
                ti="You are not veified verify yourself with 'verify' to have access to every command"
            
            
            e.title = ti

            e.description = "Note: *You do not type any **__brackets__***\n [] -> Is used for optional arguments \n<> -> Is used for required arguments\n\nSelect the category you want the help about or do `jr help command-name` if you want the help for specific command [if its a subcommand do `jr help parent-command sub-command` for example `jr help set welcome`]"
            
            await ctx.send(embed=e, view=Buttonwithselect(self.pages, self.bot))
















async def setup(bot):
    await bot.add_cog(Help(bot))
```
