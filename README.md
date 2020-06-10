# Overview:

One of the live projects that I worked on with The Tech Academy was a Hobby Tracker utilizing the Django Framework with Python where a user could utilize our app to help them track and organize their hobby. Each of us were to select a hobby and create an app to integrate in to the main Hobby Tracker app. This is a project that I was very excited to be a part of and had a lot of fun with! My chosen hobby is one that is near and dear to me; Pokemon! 

The goal of my App was to give other Pokemon Trainers a resource to help them track their Pokemon across the many games that have been released over the years. It included a several databases:

* A database user could add Pokemon, edit their details, or delete them.

* A Database to house the trainer information which was utilized to create a "Trainer Card" for them housing their details including an image for their trainer.

* A Database for the user to add and delete badges on their trainer card.

And lastly I included two features for the users for convenience. A connection to an API that would allow them to quickly search basic information about their Pokemon species and a Datascrape from a gaming news website for easy access to the latest Pokemon related news.

# Pokemon Database

    from django.db import models
    from django.core.validators import MaxValueValidator, MinValueValidator
    import datetime
    
    # The below are for keeping the data input by the user's clean.
    
    POKEMON_TYPES = (('Bug', 'Bug'), ('Dark', 'Dark'), ('Dragon', 'Dragon'), ('Electric', 'Electric'), ('Fairy', 'Fairy'),
                     ('Fighting', 'Fighting'),
                     ('Fire', 'Fire'), ('Flying', 'Flying'), ('Ghost', 'Ghost'), ('Grass', 'Grass'), ('Ground', 'Ground'),
                     ('Ice', 'Ice'),
                     ('Normal', 'Normal'), ('Poison', 'Poison'), ('Psychic', 'Psychic'), ('Rock', 'Rock'),
                     ('Steel', 'Steel'), ('Water', 'Water'), ('None', 'None'))
    
    SHINY_POKEMON = (('Yes', 'Yes'), ('No', 'No'))
    
    NUZLOCKE = (('Alive', 'Alive'), ('Fainted', 'Fainted'), ('Not Applicable', 'Not Applicable'))
    
    POKEMON_GAME = (('Red', 'Red'), ('Blue', 'Blue'), ('Yellow', 'Yellow'), ('Gold', 'Gold'), ('Silver', 'Silver'),
                    ('Crystal', 'Crystal'), ('Ruby', 'Ruby'), ('Sapphire', 'Sapphire'), ('Emerald', 'Emerald'),
                    ('Diamond', 'Diamond'), ('Pearl', 'Pearl'), ('Platinum', 'Platinum'), ('HeartGold', 'HeartGold'),
                    ('SoulSilver', 'SoulSilver'), ('Black', 'Black'), ('White', 'White'), ('Black 2', 'Black 2'),
                    ('White 2', 'White 2'), ('X', 'X'), ('Y', 'Y'), ('Omega Ruby', 'Omega Ruby'),
                    ('Alpha Sapphire', 'Alpha Sapphire'), ('Sun', 'Sun'), ('Moon', 'Moon'), ('Ultra Sun', 'Ultra Sun'),
                    ('Ultra Moon', 'Ultra Moon'), ("Let's Go Pikachu!", "Let's Go Pikachu!"),
                    ("Let's Go Eevee!", "Let's Go Eevee!"), ('Sword', 'Sword'), ('Shield', 'Shield'),
                    ('PokemonGo', 'PokemonGo'), ('Fire Red', 'Fire Red'), ('Leaf Green', 'Leaf Green'))
    
    # Pokemon Collection Database
    
    class PokemonCollection(models.Model):
        pokemon = models.CharField(max_length=25, default=None)
        game = models.CharField(max_length=100, choices=POKEMON_GAME, default=None)
        nickname = models.CharField(max_length=12, default=None)
        level = models.PositiveIntegerField(default=1, validators=[MinValueValidator(1), MaxValueValidator(100)])
        cp = models.IntegerField(default=1)
        type = models.CharField(max_length=10, choices=POKEMON_TYPES, default=None)
        subtype = models.CharField(max_length=10, choices=POKEMON_TYPES, default=None)
        shiny = models.CharField(max_length=3, choices=SHINY_POKEMON, default=None)
        nuzlocke = models.CharField(max_length=15, choices=NUZLOCKE, default=None)
        description = models.CharField(max_length=300, default=None)
    
        Pokemon = models.Manager()
    
        def __str__(self):
            return self.pokemon
        
    # Badge Collection Database
    class BadgeCollection(models.Model):
        badge = models.CharField(max_length=25, choices=BADGES, default=None)
    
        Badge = models.Manager()
    
        def __str__(self):
            return self.badge
    
    # Trainer Card Database
    class TrainerCard(models.Model):
        image = models.ImageField(upload_to='PokemonApp/')
        name = models.CharField(max_length=15, default=None)
        gender = models.CharField(max_length=15, default=None)
        age = models.IntegerField(default=	1)
    
        Name = models.Manager()
    
        def __str__(self):
            return self.name

    
# Trainer Card

    def trainer(request):
    get_badges = BadgeCollection.Badge.all()
    get_trainer = TrainerCard.Name.all()
    context = {
        'badges': get_badges,
        'trainers': get_trainer,
    }
    return render(request, 'PokemonApp/pokemon_trainer.html', context)

    def add_badge(request):
        form = BadgeForm(request.POST or None)
        if form.is_valid():
            form.save()
            return redirect('trainercard')
        else:
            print(form.errors)
            form = BadgeForm()
        return render(request, 'PokemonApp/pokemon_badgeadd.html', {'form': form})
    
    def add_trainer(request):
        form = TrainerForm(request.POST or None, request.FILES or None)
        if (len(TrainerCard.Name.all())) == 0:  # restricts the database to one name which will not allow them to add more
            # than one trainer
            if form.is_valid():
                form.save()
                return redirect('trainercard')
        else:
            print(form.errors)
            form = TrainerForm()
        return render(request, 'PokemonApp/pokemon_addtrainer.html', {'form': form})

    def delete_badge(request, pk):
        pk = int(pk)
        badge = get_object_or_404(BadgeCollection, pk=pk)
        if request.method == 'POST':
            badge.delete()
            return redirect('trainercard')
        context = {"badge": badge}
        return render(request, "PokemonApp/pokemon_badgedelete.html", context)
    
    
    def confirmed_badge(request):
        if request.method == 'POST':
            form = BadgeForm(request.POST or None)
            if form.is_valid():
                form.delete()
                return redirect('trainercard')
        else:
            return redirect('trainercard')

        # Allows the user to edit their trainer
    def edit_trainer(request, id):
        id = int(id)
        trainer = get_object_or_404(TrainerCard, id=id)
        form = TrainerForm(request.POST or None, request.FILES or None, instance=trainer)
        if form.is_valid():
            form.save()
            return redirect('trainercard')
        return render(request, 'PokemonApp/pokemon_traineredit.html', context={'form': form, })
        
displaying the trainer card

    {% extends 'PokemonApp/pokemon_base.html' %}
    {% load staticfiles %}
    {% block templatecontent %}
    <section >
        <div class="flex-container" id="pokemonCollectionPage">
            <br>
            <h3 class='pokemonBanner'>Trainer Details</h3><br><button class="primary-bright-button-pokemon" type="button" onclick=" location.href='{% url 'addTrainer' %}'">&#9683; Add your trainer details</button>
                <hr \>
            <div class="trainercard">
                {% for trainer in trainers %}
                <ul>
                <li>TRAINER NAME: {{trainer.name}}</li><br><br>
                 <li>GENDER: {{trainer.gender}}</li><br><br>
                <li>AGE: {{trainer.age}}</li><br><br>
                </ul>
                    {% if trainer.image %}
                        <img src="{{trainer.image.url}}" style="max-width:250px; max-height: 350px"><br>
                        <a href="{{trainer.id}}/EditTrainer"><button class="primary-light-button-pokemon">&#9683; Edit </button></a><br><br><!-- retrieves the image from the database and sizes it down -->
                    {% endif %}
    
                {% endfor %}
            </div>
            <br><br>
               <h3 class='pokemonBanner'>Badges Collected</h3>
            <button class="primary-bright-button-pokemon" type="button" onclick=" location.href='{% url 'addBadge' %}'">&#9683; Add Badge/Challenge to your Collection</button>
            <br><br>
             <span class="pokemonSpan">
                 <div class="pokebadgebackground">
                        <span class="badge"> <!-- The below will pull the corresponding image to the badge name to display. Researching a more clean way to do this but currently
                        have it searching for each badge name individually-->
                             {% for badges in badges %}
                            <!-- Kanto Badges -->
                                    {% if badges.badge == 'Boulder Badge' %}<img src="{% static 'images/PokemonApp/badges/boulder_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Cascade Badge' %}<img src="{% static 'images/PokemonApp/badges/cascade_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Rainbow Badge' %}<img  src="{% static 'images/PokemonApp/badges/rainbow_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Thunder Badge' %}<img src="{% static 'images/PokemonApp/badges/thunder_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Marsh Badge' %}<img src="{% static 'images/PokemonApp/badges/marsh_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Soul Badge' %}<img src="{% static 'images/PokemonApp/badges/soul_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Volcano Badge' %}<img src="{% static 'images/PokemonApp/badges/volcano_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Earth Badge' %}<img src="{% static 'images/PokemonApp/badges/earth_badge.png' %}" style="width:50px;height:50px">{% endif %}
                            <!-- Jhoto Badges -->
                                    {% if badges.badge == 'Zephyr Badge' %}<img src="{% static 'images/PokemonApp/badges/zephyr_badge.png' %}" style="width:50px;height:50px"> {% endif %}
                                    {% if badges.badge == 'Hive Badge' %}<img src="{% static 'images/PokemonApp/badges/hive_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Plain Badge' %}<img  src="{% static 'images/PokemonApp/badges/plain_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Fog Badge' %}<img src="{% static 'images/PokemonApp/badges/fog_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Storm Badge' %}<img src="{% static 'images/PokemonApp/badges/storm_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Mineral Badge' %}<img src="{% static 'images/PokemonApp/badges/mineral_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Glacier Badge' %}<img src="{% static 'images/PokemonApp/badges/glacier_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Rising Badge' %}<img src="{% static 'images/PokemonApp/badges/rising_badge.png' %}" style="width:50px;height:50px">{% endif %}
                             <!-- Hoenn Badges -->
                                    {% if badges.badge == 'Stone Badge' %}<img src="{% static 'images/PokemonApp/badges/stone_badge.png' %}" style="width:50px;height:50px"> {% endif %}
                                    {% if badges.badge == 'Knuckle Badge' %}<img src="{% static 'images/PokemonApp/badges/knuckle_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Dynamo Badge' %}<img  src="{% static 'images/PokemonApp/badges/dynamo_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Heat Badge' %}<img src="{% static 'images/PokemonApp/badges/heat_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Balance Badge' %}<img src="{% static 'images/PokemonApp/badges/balance_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Feather Badge' %}<img src="{% static 'images/PokemonApp/badges/feather_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Mind Badge' %}<img src="{% static 'images/PokemonApp/badges/mind_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Rain Badge' %}<img src="{% static 'images/PokemonApp/badges/rain_badge.png' %}" style="width:50px;height:50px">{% endif %}
                            <!-- Sinnoh Badges -->
                                    {% if badges.badge == 'Coal Badge' %}<img src="{% static 'images/PokemonApp/badges/coal_badge.png' %}" style="width:50px;height:50px"> {% endif %}
                                    {% if badges.badge == 'Forest Badge' %}<img src="{% static 'images/PokemonApp/badges/forest_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Cobble Badge' %}<img  src="{% static 'images/PokemonApp/badges/cobble_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Fen Badge' %}<img src="{% static 'images/PokemonApp/badges/fen_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Relic Badge' %}<img src="{% static 'images/PokemonApp/badges/relic_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Mine Badge' %}<img src="{% static 'images/PokemonApp/badges/mine_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Icicle Badge' %}<img src="{% static 'images/PokemonApp/badges/icicle_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Beacon Badge' %}<img src="{% static 'images/PokemonApp/badges/beacon_badge.png' %}" style="width:50px;height:50px">{% endif %}
                            <!-- Unova Badges -->
                                    {% if badges.badge == 'Trio Badge' %}<img src="{% static 'images/PokemonApp/badges/trio_badge.png' %}" style="width:50px;height:50px"> {% endif %}
                                    {% if badges.badge == 'Toxic Badge' %}<img src="{% static 'images/PokemonApp/badges/toxic_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Basic Badge' %}<img src="{% static 'images/PokemonApp/badges/basic_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Insect Badge' %}<img  src="{% static 'images/PokemonApp/badges/insect_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Bolt Badge' %}<img src="{% static 'images/PokemonApp/badges/bolt_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Quake Badge' %}<img src="{% static 'images/PokemonApp/badges/quake_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Jet Badge' %}<img src="{% static 'images/PokemonApp/badges/jet_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Freeze Badge' %}<img src="{% static 'images/PokemonApp/badges/freeze_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Legend Badge' %}<img src="{% static 'images/PokemonApp/badges/legend_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Wave Badge' %}<img src="{% static 'images/PokemonApp/badges/wave_badge.png' %}" style="width:50px;height:50px">{% endif %}
                            <!-- Kalos Badges -->
                                    {% if badges.badge == 'Bug Badge' %}<img src="{% static 'images/PokemonApp/badges/bug_badge.png' %}" style="width:50px;height:50px"> {% endif %}
                                    {% if badges.badge == 'Cliff Badge' %}<img src="{% static 'images/PokemonApp/badges/cliff_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Rumble Badge' %}<img  src="{% static 'images/PokemonApp/badges/rumble_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Plant Badge' %}<img src="{% static 'images/PokemonApp/badges/plant_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Voltage Badge' %}<img src="{% static 'images/PokemonApp/badges/voltage_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Fairy Badge Kalos' %}<img src="{% static 'images/PokemonApp/badges/fairykalos_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Psychic Badge' %}<img src="{% static 'images/PokemonApp/badges/psychic_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Iceberg Badge' %}<img src="{% static 'images/PokemonApp/badges/iceberg_badge.png' %}" style="width:50px;height:50px">{% endif %}
                            <!-- Galar Badges -->
                                    {% if badges.badge == 'Grass Badge' %}<img src="{% static 'images/PokemonApp/badges/grass_badge.png' %}" style="width:50px;height:50px"> {% endif %}
                                    {% if badges.badge == 'Water Badge' %}<img src="{% static 'images/PokemonApp/badges/water_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Fire Badge' %}<img  src="{% static 'images/PokemonApp/badges/fire_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Fighting Badge' %}<img src="{% static 'images/PokemonApp/badges/fighting_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Ghost Badge' %}<img src="{% static 'images/PokemonApp/badges/ghost_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Fairy Badge Galar' %}<img src="{% static 'images/PokemonApp/badges/fairygalar_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Rock Badge' %}<img src="{% static 'images/PokemonApp/badges/rock_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Ice Badge' %}<img src="{% static 'images/PokemonApp/badges/ice_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Dark Badge' %}<img src="{% static 'images/PokemonApp/badges/dark_badge.png' %}" style="width:50px;height:50px">{% endif %}
                                    {% if badges.badge == 'Dragon Badge' %}<img src="{% static 'images/PokemonApp/badges/dragon_badge.png' %}" style="width:50px;height:50px">{% endif %}
                            {% endfor %}
                        </span>
                 </div>
                 <a href="{% url 'listBadges' %}"><button class="primary-light-button-pokemon">&#9683; Delete</button></a><br><br>
             </span>
    
            <!-- for trials and kahunas -->
            <h3 class='pokemonBanner'>Elite 4 Champions Defeated</h3>
            <span class="pokemonSpan">
                 <div class="pokebadgebackground">
                        <span class="badge"> <!-- The below will pull the corresponding image to the badge name to display. Researching a more clean way to do this but currently
                        have it searching for each badge name individually-->
                             {% for badges in badges %}
                            <!-- Island Captains -->
                                    {% if badges.badge == 'Champion Blue' %}<img src="{% static 'images/PokemonApp/badges/champion_blue.png' %}" style="width:135px;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Trace' %}<img src="{% static 'images/PokemonApp/badges/champion_trace.png' %}" style="width:135px;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Lance' %}<img src="{% static 'images/PokemonApp/badges/champion_lance.png' %}" style="width:135px;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Red' %}<img  src="{% static 'images/PokemonApp/badges/champion_red.png' %}" style="width:135px;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Steven' %}<img src="{% static 'images/PokemonApp/badges/champion_steven.png' %}" style="width:135px;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Wallace' %}<img src="{% static 'images/PokemonApp/badges/champion_wallace.png' %}" style="width:135px;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Cynthia' %}<img src="{% static 'images/PokemonApp/badges/champion_cynthia.png' %}" style="width:250135;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Alder' %}<img src="{% static 'images/PokemonApp/badges/champion_alder.png' %}" style="width:135px;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Iris' %}<img src="{% static 'images/PokemonApp/badges/champion_iris.png' %}" style="width:175px;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Diantha' %}<img src="{% static 'images/PokemonApp/badges/champion_diantha.png' %}" style="width:135px;height:225px">{% endif %}
                                    {% if badges.badge == 'Champion Leon' %}<img src="{% static 'images/PokemonApp/badges/champion_leon.png' %}" style="width:135px;height:225px">{% endif %}
                                    {% if badges.badge == 'Professor Kukui' %}<img src="{% static 'images/PokemonApp/badges/professor_kukui.png' %}" style="width:135px;height:225px">{% endif %}
                            {% endfor %}
                        </span>
                 </div>
             </span>
            <h3 class='pokemonBanner'>Trials Completed</h3>
            <span class="pokemonSpan">
                 <div class="pokebadgebackground">
                        <span class="badge"> <!-- The below will pull the corresponding image to the badge name to display. Researching a more clean way to do this but currently
                        have it searching for each badge name individually-->
                             {% for badges in badges %}
                            <!-- Island Captains -->
                                    {% if badges.badge == 'Captain Lima' %}<img src="{% static 'images/PokemonApp/badges/trial_captain_lima.png' %}" style="width:250px;height:130px">{% endif %}
                                    {% if badges.badge == 'Captain Kiawe' %}<img src="{% static 'images/PokemonApp/badges/trial_captain_kiawe.png' %}" style="width:250px;height:130px">{% endif %}
                                     {% if badges.badge == 'Captain Lana' %}<img src="{% static 'images/PokemonApp/badges/trial_captain_lana.png' %}" style="width:250px;height:130px">{% endif %}
                                    {% if badges.badge == 'Captain Mallow' %}<img  src="{% static 'images/PokemonApp/badges/trial_captain_mallow.png' %}" style="width:250px;height:130px">{% endif %}
                                    {% if badges.badge == 'Captain Sophocles' %}<img src="{% static 'images/PokemonApp/badges/trial_captain_sophocles.png' %}" style="width:250px;height:130px">{% endif %}
                                    {% if badges.badge == 'Captain Acerola' %}<img src="{% static 'images/PokemonApp/badges/trial_captain_acerola.png' %}" style="width:250px;height:130px">{% endif %}
                                    {% if badges.badge == 'Captain Mina' %}<img src="{% static 'images/PokemonApp/badges/trial_captain_mina.png' %}" style="width:250px;height:130px">{% endif %}
                            <!-- Island Kahunas -->
                                    {% if badges.badge == 'Akala Kahuna' %}<img src="{% static 'images/PokemonApp/badges/Akala_Trialpng.png' %}" style="width:128px;height:128px"> {% endif %}
                                    {% if badges.badge == 'Poni Kahuna' %}<img src="{% static 'images/PokemonApp/badges/Poni_Trial.png' %}" style="width:128px;height:128px">{% endif %}
                                    {% if badges.badge == 'Ulaula Kahuna' %}<img  src="{% static 'images/PokemonApp/badges/Ulaula_Trial.png' %}" style="width:128px;height:128px">{% endif %}
                                    {% if badges.badge == 'Melemele Kahuna' %}<img src="{% static 'images/PokemonApp/badges/Melemele_Trial.png' %}" style="width:128px;height:128px">{% endif %}
                            {% endfor %}
                        </span>
                 </div>
                <a href="{% url 'listBadges' %}"><button class="primary-light-button-pokemon">&#9683; Delete</button></a>
             </span>
        </div>
        <div class="pokemonnews">
            <br><a href="#section1">Return to the top of the page</a>
        </div>
    </section>
    {% endblock %}


# The Pokemon Index Page

    # Renders the page that lists all of the pokemon in the database
    def index(request):
        get_pokemon = PokemonCollection.Pokemon.all()  # Gets all of the Pokemon from the PokemonCollection model.
        context = {'pokemon': get_pokemon}
        return render(request, 'PokemonApp/pokemon_indexpage.html', context)  # brings up the pokemon_indexpage for the user

    # Renders the page that allows you to add a pokemon to the database
    def add_pokemon(request):
        form = PokemonForm(request.POST or None)
        if form.is_valid():  # validates the form and then saves in to the database.
            form.save()
            return redirect('listPokemon')  # returns the user to the database
        else:
            print(form.errors)
            form = PokemonForm()
        return render(request, 'PokemonApp/pokemon_create.html', {'form': form})  # brings up the page for adding a badge
    # to the database for the user
    
        # Renders the details page for the selected pokemon
    def details_pokemon(request, pk):
        pk = int(pk)
        pokemon = get_object_or_404(PokemonCollection, pk=pk)  # references the PokemonCollection model and looks up
        context = {'pokemon': pokemon}  # the user's selected pokemon.
        return render(request, 'PokemonApp/pokemon_details.html', context)  # brings up the details page for the selected
    # pokemon
    
    
    # Allows the user to select a pokemon for deletion from the Database and renders the confirm delete page
    def delete_pokemon(request, pk):
        pk = int(pk)
        pokemon = get_object_or_404(PokemonCollection, pk=pk)  # references the PokemonCollection model
        if request.method == 'POST':
            pokemon.delete()
            return redirect('listPokemon')  # returns the user to the database list
        context = {"pokemon": pokemon}
        return render(request, "PokemonApp/pokemon_confirmdelete.html", context)  # Brings the user to the confirm delete
    # page
    
    
    # Allows the user to confirm the deletion and then returns the user to the database list.
    def confirmed(request):
        if request.method == 'POST':
            form = PokemonForm(request.POST or None)
            if form.is_valid():  # validates the form and deletes from the deatabase
                form.delete()
                return redirect('listPokemon')  # returns the user to the pokemon database list.
        else:
            return redirect('listPokemon')
            
         # Allows the user to select a pokemon to edit from the Database and renders the edit page
    def edit_pokemon(request, id):
        id = int(id)  # references the ID for the selected pokemon
        pokemon = get_object_or_404(PokemonCollection, id=id)  # references the PokemonCollection database for the ID
        form = PokemonForm(request.POST or None, instance=pokemon)
        if form.is_valid():  # validates the form and then saves the user's changes
            form.save()
            return redirect('listPokemon')  # returns the user to the pokemon database list.
        return render(request, 'PokemonApp/pokemon_edit.html', context={'form': form, })  # Brings up the edit page for the
    # user

displaying the information in the database

    {% extends 'PokemonApp/pokemon_base.html' %}
    {% load staticfiles %}
    {% block templatecontent %}
    <section >
        <div class="flex-container" id="pokemonCollectionPage">
            <br>
            <div id="pokemonPic">
                <img src="{% static 'images/PokemonApp/chansey_transparent.png' %}" style="width:300px;height:250px">
            </div>
            <div class="pokebackground">
                <span class="pokemonSpan">
                    Welcome to the Pokemon App Database! Click on "Add Pokemon to your Collection" to begin building your
                    collection.<br>
                    <br>
                    From there you can Delete and Edit your entries using the "options" column or click on "Details"
                    to pull up a full description of your pokemon.
                </span>
            </div>
            <br>
            <table class="pokemonAddTable">
                <tr>
                    <th class="col-md">Pokemon</th>
                    <th class="col-md">Game</th>
                    <th class="col-md">Nickname</th>
                    <th class="col-md">Level</th>
                    <th class="col-md">CP</th>
                    <th class="col-md">Type</th>
                    <th class="col-md">SubType</th>
                    <th class="col-md">Shiny</th>
                    <th class="col-md">Nuzlocke</th>
                    <th class="col-md">Options</th>
                </tr>
                {% for pokemon in pokemon %}
                    <tr>
                        <td class="col-md">{{pokemon.pokemon}} {% if pokemon.shiny == 'Yes' %}&#127775;{% endif %}
                            {% if pokemon.nuzlocke == 'Fainted' %}&#128577;{% endif %}
                            {% if pokemon.nuzlocke == 'Alive' %}&#128512;{% endif %}
                        </td>
                        <td class="col-md">{{pokemon.game}}</td>
                        <td class="col-md">{{pokemon.nickname}}</td>
                        <td class="col-md">{{pokemon.level}}</td>
                        <td class="col-md">{{pokemon.cp}}</td>
                        <td class="col-md">{{pokemon.type}}</td>
                        <td class="col-md">{{pokemon.subtype}}</td>
                        <td class="col-md">{{pokemon.shiny}}</td>
                        <td class="col-md">{{pokemon.nuzlocke}}</td>
                        <td class="col-md"><a href="{{pokemon.pk}}/Details"><button class="primary-light-button-pokemon">&#9683; Details </button></a><br><br>
                            <a href="{{pokemon.pk}}/DeletePokemon"><button class="primary-light-button-pokemon">&#9683; Delete </button></a><br><br>
                            <a href="{{pokemon.id}}/EditPokemon"><button class="primary-light-button-pokemon">&#9683; Edit </button></a><br><br>
                        </td>
    
                    </tr>
                {% endfor %}
            </table>
            <button class="primary-bright-button-pokemon" type="button" onclick=" location.href='{% url 'addPokemon' %}'">&#9683; Add Pokemon to your Collection</button>
        </div>
        <div class="pokemonnews">
            <br><a href="#section1">Return to the top of the page</a>
        </div>
    </section>
    {% endblock %}
    

displaying the pokemon details

    {% extends 'PokemonApp/pokemon_base.html' %}
    {% load staticfiles %}
    {% block templatecontent %}
    <section>
        <div class="flex-container" id="pokemonDetailsPage">
            <br>
            <div id="pokemonPic">
                <img src="{% static 'images/PokemonApp/pokeball_transparent.png' %}" style="width:250px;height:250px">
            </div>
            <table class="pokemoncenter">
                <tr>
                    <th>Pokemon</th>
                    <td>{{pokemon.pokemon}}</td>
                </tr>
                <tr>
                    <th>Game</th>
                    <td>{{pokemon.game}}</td>
                </tr>
                <tr>
                    <th>Nickname</th>
                    <td>{{pokemon.nickname}}</td>
                </tr>
                <tr>
                    <th>Level</th>
                    <td>{{pokemon.level}}</td>
                </tr>
                <tr>
                    <th>CP</th>
                    <td>{{pokemon.cp}}</td>
                </tr>
                <tr>
                    <th>Type</th>
                    <td>{{pokemon.type}}</td>
                </tr>
                <tr>
                    <th>SubType</th>
                    <td>{{pokemon.subtype}}</td>
                </tr>
                <tr>
                    <th>Shiny</th>
                    <td>{{pokemon.Shiny}}</td>
                </tr>
                <tr>
                    <th>Nuzlocke</th>
                    <td>{{pokemon.nuzlocke}}</td>
                </tr>
                <tr>
                    <th>Description</th>
                    <td>{{pokemon.description}}</td>
                </tr>
    
            </table>
            <hr />
            <button class="primary-bright-button-pokemon" type="button" onclick=" location.href='{% url 'listPokemon' %}'">&#9683; Return to List</button>
        </div>
    </section>
    {% endblock %}



# Connecting to and Parsing through the API

        # Allows the user to search for a pokemon and pull up some basic details by connecting to the PokeAPI.
    def pokemon_api_search(request):
        try:
            if request.method == 'POST':
                searched_pokemon = request.POST['searched_pokemon']  # the user's searched pokemon.
                pokemon_search = requests.get(f"https://pokeapi.co/api/v2/pokemon/{searched_pokemon.lower()}")  # connects
                # to the API with the user's searched pokemon forced in to lowercase.
                pokemon_found = json.loads(pokemon_search.content)  # loads the found pokemon from the json
                # creating empty lists that will be added to context so they can be referenced and displayed as data
                # on the html file
                abilities = []
                pokedexentry = []
                stats = []
                types = []
                moves = []
                for ability in range(0, len(pokemon_found['abilities'])):  # search through all abilities in the json
                    abilities.append(pokemon_found['abilities'][ability]['ability']['name'])  # append the empty list
                for pokedex in range(0, len(pokemon_found['game_indices'])):  # search through all game indexes in the json
                    pokedexentry.append(pokemon_found['game_indices'][pokedex]['version']['name'])  # append the empty list
                    pokedexentry.append(pokemon_found['game_indices'][pokedex]['game_index'])  # append the empty list
                    # this list is appended twice, once for the version name and again for the index #
                for stat in range(0, len(pokemon_found['stats'])):  # search through all stats in the json
                    stats.append(pokemon_found['stats'][stat]['stat']['name'])  # append the empty list
                    stats.append(pokemon_found['stats'][stat]['base_stat'])  # append the empty list
                    # this list is appended twice, once for the stat name and again for the stat #
                for move in range(0, len(pokemon_found['moves'])):  # append the empty list
                    moves.append(pokemon_found['moves'][move]['move']['name'])  # search through all abilities in the json
                for type in range(0, len(pokemon_found['types'])):  # search through all types in the json
                    types.append(pokemon_found['types'][type]['type']['name'])  # append the empty list
                context = {
                    'searched_pokemon': searched_pokemon,
                    'pokemon_found': pokemon_found,
                    'abilities': abilities,
                    'pokedexentry': pokedexentry,
                    'stats': stats,
                    'types': types,
                    'moves': moves,
                }
                return render(request, 'PokemonApp/pokemon_api.html', context)  # renders the pokemon API page with the
                # searched and found pokemon in context.
            else:
                return render(request, 'PokemonApp/pokemon_api.html')
        except:  # if the try fails do the below:
            not_found = 'The searched pokemon was not found. Please try again.'
            context = {'not_found': not_found}
            return render(request, 'PokemonApp/pokemon_api.html', context)

displaying the API results

    {% extends 'PokemonApp/pokemon_base.html' %}
    {% load staticfiles %}
    {% block templatecontent %}
    <section>
        <div class="flex-container" id="pokemonDetailsPage">
            <div id="pokemonPic">
                <img src="{% static 'images/PokemonApp/eevee_transparent.png' %}" style="width:300px;height:300px">
            </div>
            <div class="pokebackground">
                <span class="pokemonSpan">
                   Unsure about the details on a pokemon you want to enter in to the database?<br>
                    Search the name of the Pokemon you want to add to pull up their basic details!
                </span>
            </div>
            <form method="POST" action="{% url 'pokemonApi' %}"> <!-- posts to this page-->
              {% csrf_token %}
              <input type="search" placeholder="Search for a Pokemon" name="searched_pokemon"> <!-- allows the user to search for a pokemon-->
              <button class="primary-bright-button-pokemon" type="submit">Search pokemon</button>
            </form>
            <br/>
            <table class="pokemoncenter">
                <tr>
                    <th>Pokemon</th>
                    <td>{{pokemon_found.name}}</td>
                </tr>
                <tr>
                    <th>Index <Number></Number></th>
                    <td>{% for pokedexentry in pokedexentry %}{{pokedexentry}}, {% endfor %}</td> <!-- reference the pokedexentry list created from the json for the searched pokemon-->
                </tr>
                 <tr>
                    <th>Height and Weight</th>
                    <td>Height:{{pokemon_found.height}}<br> Weight: {{pokemon_found.weight}}</td>
                </tr>
                <tr>
                <tr>
                    <th>Abilities</th>
                    <td>
                        <ol class="pokelist">{% for abilities in abilities %} <li>{{abilities}}</li> <!-- reference the abilities list created from the json for the searched pokemon
                        and puts them in a list style-->
                        {% endfor %}</ol>
                    </td>
                </tr>
                    <th>Base XP</th>
                    <td>{{pokemon_found.base_experience}}</td>
                </tr>
                <tr>
                    <th>Base Stat total</th>
                    <td>{% for stats in stats %}{{stats}}<br>{% endfor %}</td> <!-- reference the base stat total list created from the json for the searched pokemon-->
                </tr>
                <tr>
                    <th>Types</th>
                    <td>{% for types in types %}{{types}}<br>{% endfor %}</td> <!-- reference the types list created from the json for the searched pokemon-->
                </tr>
                <tr>
                    <th>Moves</th>
                    <td>{% for moves in moves %}{{moves}}, {% endfor %}</td> <!-- reference the moves list created from the json for the searched pokemon-->
                </tr>
            </table>
            <br/>
        </div>
        <div class="pokemonnews">
            <br><a href="#section1">Return to the top of the page</a>
        </div>
    </section>
    {% endblock %}

# DataScrape for the News Page	

    # datascrape to display news info
    def pokemon_news(request):
        page = requests.get('https://www.gamespot.com/search/?i=articles&q=pokemon')  # requests html doc from gamespot
        soup = BS(page.content, 'html.parser')
        articles = soup.find_all(class_="media-figure media-figure--search")  # searches for the tag
        # media-figure media-figure--search from the html file and stores it in articles
        pages = []  # empty pages array to store data
        for article in articles:
            title = article.find('img').get('alt')  # searches through articles for the alt tag in img
            link = article.find('a').get('href')  # searches through articles for the a tag in href
            img = article.find('img').get('src')  # searches through articles for the img tag in src
            url = "https://www.gamespot.com" + link  # creates the url as the base url for gamespot + the link we created
            article = {'title': title, 'url': url, 'img': img, }  # stores title, img, and url to be added to the array
            pages.append(article)  # appends our array
        context = {'pages': pages}  # puts them in to context to be referenced in the html file
        return render(request, 'PokemonApp/pokemon_news.html', context)  #renders the page with the context
    
displaying the results of the datascrape

    {% extends 'PokemonApp/pokemon_base.html' %}
    {% load staticfiles %}
    {% block templatecontent %}
    <section>
            <div class="flex-container" id="pokemonDetailsPage">
                <div id="pokemonPic">
                    <img src="{% static 'images/PokemonApp/tyranitar_transparent.png' %}" style="width:400px;height:400px">
                </div>
                <div class="pokebackground">
                    <span class="pokemonSpan">
                        Stay up to date on all Pokemon News! Below are a list of the most recent articles about Pokemon
                        pulled from gamespot. Click the links below the article to go directly to gamespot to read the article!<br><br>
                        Article will open up in a new tab.
                    </span>
                    <br><br>
                </div>
                    <div class="pokemonnews">
                    {% for page in pages %}
                        <h4>{{page.title}}</h4><br>
                        <img src="{{page.img}}" alt="" style="width:250px;height:250px"><br><br>
                        <p><a href="{{page.url}}" target="_blank" >Link to the original Article</a></p>
                        <br/>
                    {% endfor %}
                        <br><a href="#section1">Return to the top of the page</a>
                    </div>
            </div>
    </section>
    {% endblock %}
    

# What I've Learned:

This project was a great experience for owning the end to end design of the application and was really able to use my creativity to create a product that I was proud of. I was also able to hone valuable skills while learning to work with:
* Python
* Django
* APIs
* Data Scraping
* Json
* HTML
* CSS

I enjoyed working with my live project team and even though we were all working on unique applications as part of the bigger Hobby Tracker Application we were able to work as a team to discuss what we were working on, roadblocks or troubles we were having while learning from one anothers troubles and mistakes, and sharing tips and tricks we picked up along the way. 
