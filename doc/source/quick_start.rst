Quick start
===========

Follow the install and configure instructions.

With Django sampledatahelper you have 2 options to populate your database

Using SampleDataFiller
----------------------

Sample data filler is a command that use the ``SAMPLEDATAHELPER_MODELS`` setting
variable to populate your database. Example::

  SAMPLEDATAHELPER_MODELS = [
      # Generate 5 instances completly random
      { 'model': 'myapp.MyModel', 'number': 5, },

      # Generate 5 instances selecting random method for some fields
      {
          'model': 'myapp.MyModel',
          'number': 5,
          'fields_overwrite': [
              ('my_int_field', lambda _, sd: sd.int(5, 10)),
          ]
      },

      # Generate 5 instances with fixed data in a field
      {
          'model': 'myapp.MyModel',
          'number': 5,
          'fields_overwrite': [
              ('my_int_field', 5),
          ]
      }
  ]

Then you only have to run::

  python manage.py sampledatafiller

Using a custom sampledata command
---------------------------------

You can create a command to fill your models manullay to take more control.

If you have some aplications to populate, you can split your sample data
generation on one command per app, or add only one command in one app thats
generate everything.

The file must be in ``<app-module>/management/commands/<command-name>.py`` can be
something like ``myapp/management/commands/mysampledata.py``.

The easy way to build your command is using :py:class:`ModelDataHelper`::

  from django.core.management.base import BaseCommand
  from myapp.models import MyModel
  from sampledatahelper.model_helper import ModelDataHelper
  from sampledatahelper.helper import SampleDataHelper
  
  class Command(BaseCommand):
      args = ''
      help = 'Example data generator'
      mdh = ModelDataHelper(seed=12345678901)
  
      def handle(self, *args, **options):
          print "Generating MyModel data"
          # Generate 5 instances completly random
          self.mdh.fill_model(MyModel, 5)
  
          # Generate 5 instances selecting random method for some fields
          self.mdh.fill_model(MyModel,
                              5,
                              my_int_field=lambda instance, sd: sd.int(5, 10))
  
          # Generate 5 instances with fixed data in a field
          self.mdh.fill_model(MyModel, 5, my_int_field=8)

You can build a more precise command using directly the :py:class:`SampleDataHelper`::

  from django.core.management.base import BaseCommand
  from myapp.models import MyModel
  from sampledatahelper.helper import SampleDataHelper
  
  class Command(BaseCommand):
      args = ''
      help = 'Example data generator'
      sd = SampleDataHelper(seed=12345678901)
  
      def generate_mymodel_data(self, instances):
          for x in range(instances):
              instance = MyModel.objects.create(
                  slug=self.sd.slug(2, 3),
                  name=self.sd.name(2, 3),
                  claim=self.sd.sentence(),
                  description=self.sd.paragraph(),
                  email=self.sd.email(),
                  photo=self.sd.image(64, 64),
                  is_active=self.sd.boolean(),
                  birth_date=self.sd.past_date(),
                  expected_death_date=self.sd.future_date(),
                  my_related_object=self.sd.db_object(MyRelatedModel)
              )
  
      def handle(self, *args, **options):
          print "Generating MyModel data"
          self.generate_mymodel_data(5)

To generate your sampledata, simply run the created command, for example::

  python manage.py mysampledata

