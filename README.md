# Prosper IT Consulting Internship

## Introduction
I spent a month interning with Propser IT Consulting working on two seperate projects with two seperate teams. This repository is a monument to what I achieved during my time with them.

The first half of the internship had me working on a team where each team member developed a web app from scratch that utilized a RESTful API and was linked to a hub site. The work environment was AGILE-based and tasks were served through Azure DevOps in the form of stories. The project programming language and framework were Python and Django. Each story served to me was a sequential set of features the web app was to have but otherwise I was free to develop whatever kind of web app I would like and choose whatever API I'd like to work with. I chose to develop a comic database/review website using the ComicVine API. The fruits of this project will be housed in this repository after some fine-tuning.

For the last half of my month-long internship with Prosper IT Consulting I worked on a team developing a website for a theatre troupe with a content management system. The work environment for this project was also AGILE-based and similarly tasks were served through Azure DevOps in the form of stories. The project programming language and framework were C# and ASP.NET (.NET Framework) using MVC/MVVM design patterns. The team was allowed to pick up any single story they, including myself, felt like working on at a time and I personally chose to primarily work on the administrator dashboard of the content management system. All in all, the internship was a great opportunity to learn and to contribute to the project in a meaningful way.

Below is a summary of what I worked on during the second half of the internship including descriptions of the stories, my thought process and code snippets.

## Stories
* [Log-in Exception Handling](#log-in-exception-handling)
* [Models Without Photos](#models-without-photos)
* [Photo Upload Modal](#photo-upload-modal)

### Log-in Exception Handling
The first story I worked on involved log-in exception handling. The code checks to see if the email provided by the user is linked to an existing account in the database by setting a variable equal to the response from the UserManager class' FindByEmail method which returns a user model.

      var tempUser = UserManager.FindByEmail(model.Email);


If the variable is equal to null then the account doesn't exist and the user is sent to the log-in failure view. Otherwise the user's log-in attempt is checked against a few cases to find out whether the attempt is successful, whether the account has been locked out for whatever reason (i.e. too many failed log-in attempts), whether the account needs email verification or whether the log-in attempt was an outright failure (i.e. wrong password). 

      if (tempUser != null)
      {
          string tempUserName = tempUser.UserName;
          var result = await SignInManager.PasswordSignInAsync(tempUserName, model.Password,
          model.RememberMe, shouldLockout: false);

          switch (result)
          {
              case SignInStatus.Success:
                  return RedirectToLocal(returnUrl);
              case SignInStatus.LockedOut:
                  return View("Lockout");
              case SignInStatus.RequiresVerification:
                  return RedirectToAction("SendCode", new { ReturnUrl = returnUrl, RememberMe = model.RememberMe });
              case SignInStatus.Failure:
                  return View("LoginFailure");
              default:
                  ModelState.AddModelError("", "Invalid login attempt.");
                  return View(model);
          }
      }
      else
      {
        return View("LoginFailure");
      }

### Models Without Photos
The second story I worked on involved creating a section in the administrator dashboard for any models without photos attached to them to be listed. The intent behind this story was to make the administrator aware of any models without photos and provide them with options for what the administrator wanted to do with said models whether it be to edit the model or delete it outright.


The code snippet below contains the models without photos section I created in the administrator dashboard view.

      <div id="dashboard-section-container">
          <h4 class="my-3" style="color: var(--secondary-color);">Models Without Photos</h4>

          @{
              if (Model.models_missing_photos.Productions.Count() > 0)
              {
                  <p class="h5">Productions:</p>
                  foreach (var production in (List<TheatreCMS.Models.Production>)ViewData["ProductionsWithoutPhotos"])
                  {
                      <p class="dashboard-subsection">
                          @production.Title
                          <span class="dashboard-badges mx-3">
                              @Html.ActionLink("Edit", "Edit", "Productions",
                                  new { id = production.ProductionId }, new { @class = "badge badge-pill" }) |
                              @Html.ActionLink("Delete", "Delete", "Productions",
                                  new { id = production.ProductionId }, new { @class = "badge badge-pill" })
                          </span>
                      </p>
                  }
              }

              if (Model.models_missing_photos.ProductionPhotos.Count() > 0)
              {
                  <p class="h5">Production Photos:</p>
                  foreach (var prodPhotos in (List<TheatreCMS.Models.ProductionPhotos>)ViewData["ProductionPhotosWithoutPhotos"])
                  {
                      <p class="dashboard-subsection">
                          @prodPhotos.Title
                          <span class="dashboard-badges mx-3">
                              @Html.ActionLink("Edit", "Edit", "ProductionPhotos",
                                  new { id = prodPhotos.ProPhotoId }, new { @class = "badge badge-pill" }) |
                              @Html.ActionLink("Delete", "Delete", "ProductionPhotos",
                                  new { id = prodPhotos.ProPhotoId }, new { @class = "badge badge-pill" })
                          </span>
                      </p>
                  }
              }

              if (Model.models_missing_photos.Sponsors.Count() > 0)
              {
                  <p class="h5">Sponsors:</p>
                  foreach (var sponsors in (List<TheatreCMS.Models.Sponsor>)ViewData["SponsorsWithoutPhotos"])
                  {
                      <p class="dashboard-subsection">
                          @sponsors.Name
                          <span class="dashboard-badges mx-3">
                              @Html.ActionLink("Edit", "Edit", "Sponsors",
                                  new { id = sponsors.SponsorId }, new { @class = "badge badge-pill" }) |
                              @Html.ActionLink("Delete", "Delete", "Sponsors",
                                  new { id = sponsors.SponsorId }, new { @class = "badge badge-pill" })
                          </span>
                      </p>
                  }
              }

              if (Model.models_missing_photos.CastMembers.Count() > 0)
              {
                  <p class="h5">Cast Members:</p>
                  foreach (var castMembers in (List<TheatreCMS.Models.CastMember>)ViewData["CastMembersWithoutPhotos"])
                  {
                      <p class="dashboard-subsection">
                          @castMembers.Name
                          <span class="dashboard-badges mx-3">
                              @Html.ActionLink("Edit", "Edit", "CastMembers",
                                  new { id = castMembers.CastMemberID }, new { @class = "badge badge-pill" }) |
                              @Html.ActionLink("Delete", "Delete", "CastMembers",
                                  new { id = castMembers.CastMemberID }, new { @class = "badge badge-pill" })
                          </span>
                      </p>
                  }
              }
          }
      </div>

There's some razor syntax utilized here to determine whether or not to display anything at all in the dashboard by sorting through all relevant models (Production, ProductionPhotos, Sponsor, CastMember) to find models that have no photos linked to them. If none are found then nothing is displayed. Razor syntax is also used to create pill buttons linked to controller actions, or methods, for editing the relevant models' properties or deleting the model outright.


The code snippet below contains a collection of controller actions that I created that return a list of the relevant models without photos (Production, ProductionPhotos, Sponsor, CastMember).

        public static List<CastMember> GetCastMembersWithoutPhotos()
        {
            var admin = new AdminController();
            List<CastMember> castMembersWithoutPhotos = admin.db.CastMembers.
                Where(p => p.PhotoId == null).
                OrderBy(p => p.CastMemberID).ToList();
            return castMembersWithoutPhotos;
        }
        public static List<Production> GetProductionsWithoutPhotos()
        {
            var admin = new AdminController();
            List<Production> productionsWithoutPhotos = admin.db.Productions.
                Where(p => p.DefaultPhoto == null || p.DefaultPhoto.Title == "Photo Unavailable").
                OrderBy(p => p.ProductionId).ToList();
            return productionsWithoutPhotos;
        }
        public static List<ProductionPhotos> GetProductionPhotosWithoutPhotos()
        {
            var admin = new AdminController();
            List<ProductionPhotos> productionPhotosWithoutPhotos = admin.db.ProductionPhotos.
                Where(p => p.PhotoId == null || p.Title == "Photo Unavailable" && p.Production != null).
                OrderBy(p => p.ProPhotoId).ToList();
            return productionPhotosWithoutPhotos;
        }
        public static List<Sponsor> GetSponsorsWithoutPhotos()
        {
            var admin = new AdminController();
            List<Sponsor> sponsorsWithoutPhotos = admin.db.Sponsors.
                Where(p => p.LogoId == null).
                OrderBy(p => p.LogoId).ToList();
            return sponsorsWithoutPhotos;
        }
        
This collection of controller actions is then used to pass the lists of models without photos for display to the dashboard view using a method called AddViewData as presented in the code snippet below.

        public void AddViewData(AdminSettings currentSettings)
        {
            currentSettings = AdminSettingsReader.CurrentSettings();
            List<SelectListItem> productionList = GetSelectListItems();
            #region ViewData
            ViewData["ProductionList"] = productionList;
            ViewData["CurrentProductionList"] = GetCurrentProductions();
            ViewData["nextSeasonNotification"] = NextSeasonNotification();
            ViewData["CastMembersWithoutPhotos"] = GetCastMembersWithoutPhotos();
            ViewData["ProductionsWithoutPhotos"] = GetProductionsWithoutPhotos();
            ViewData["ProductionPhotosWithoutPhotos"] = GetProductionPhotosWithoutPhotos();
            ViewData["SponsorsWithoutPhotos"] = GetSponsorsWithoutPhotos();
            #endregion
        }

The ViewData method, from the ViewDataDictionary class of the System.Web.Mvc namespace, gets and sets the dictionary for the specified view data, meaning specific data to be passed to the view. In this case I used this method to set the "CastMembersWithoutPhotos", "ProductionsWithoutPhotos", "ProudctionPhotosWithoutPhotos" and the "SponsorsWithoutPhotos" view data. I also used this method to get the view data necessary for display in the Models Without Photos section of the dashboard view.


The code snippet below is an example portion of the dashboard view demonstrating this in the parameters of the foreach loop.

        foreach (var production in (List<TheatreCMS.Models.Production>)ViewData["ProductionsWithoutPhotos"])
        {
            <p class="dashboard-subsection">
                @production.Title
                <span class="dashboard-badges mx-3">
                    @Html.ActionLink("Edit", "Edit", "Productions",
                        new { id = production.ProductionId }, new { @class = "badge badge-pill" }) |
                    @Html.ActionLink("Delete", "Delete", "Productions",
                        new { id = production.ProductionId }, new { @class = "badge badge-pill" })
                </span>
             </p>
        }

The code snippet below is a controller method that existed prior to me working on the project that I modified for the story.


The content management system handles missing Production and ProductionPhotos photos by inserting a stock "Photo Unavailable" photo where models have no photo. I modified the logic to account for this photo by its title and not it's ID because the ID could potentially change if the model for the "Photo Unavailable" photo somehow got shuffled in the database. I'm unsure if this is a full proof solution, in fact I'm pretty sure it isn't, but it remedied  the issues I could think of at the time I was working on the project. Most likely the "Photo Unavailable" photo will need to be flagged as uneditable in some way but this task wasn't a part of my story so I simply made a note of it and continued with the task at hand.
        
        public static ModelsWithoutPhotos FindModelsNoPics()
        {
            var admin = new AdminController();
            
            int photoUnavailable = admin.db.Photo.
            Where(p => p.Title == "Photo Unavailable").
            Select(p => p.PhotoId).FirstOrDefault();
            
            ModelsWithoutPhotos modelsWithNoPics = new ModelsWithoutPhotos
            {
                CastMembers = admin.db.CastMembers.
                Where(p => p.PhotoId == null).
                OrderBy(p => p.CastMemberID).
                Select(p => p.CastMemberID).ToList(),

                Productions = admin.db.Productions.
                Where(p => p.DefaultPhoto == null || p.DefaultPhoto.PhotoId == photoUnavailable).
                OrderBy(p => p.ProductionId).
                Select(p => p.ProductionId).ToList(),

                ProductionPhotos = admin.db.ProductionPhotos.
                Where(p => p.PhotoId == null || p.PhotoId == photoUnavailable && p.Production != null).
                OrderBy(p => p.ProPhotoId).
                Select(p => p.ProPhotoId).ToList(),

                Sponsors = admin.db.Sponsors.
                Where(p => p.LogoId == null).OrderBy(p => p.LogoId).
                Select(p => p.SponsorId).ToList()
            };

            return modelsWithNoPics;
        }

There was also some minor CSS I wrote for the section to keep the aesthetic of the site cohesive as necessitated by the project lead.

### Photo Upload Modal
The third story I worked on involved providing a third option for the administrator when managing models in the models without photos section of the dashboard. I was to do this by creating a third pill button that links to a pop-up modal, using JavaScript/jQuery, that allows the admin to upload a photo from their PC for the specified model missing a photo.

The code snippet below contains the code for displaying the upload photo pill button and the tags needed for JQuery to access the necessary model data.

        <a class="badge badge-pill production-modal-button" data-id=@production.ProductionId 
        data-title=@production.Title data-toggle="modal" href="#add-production-photo">Add Photo</a>
        
        <a class="badge badge-pill prod-photo-modal-button" data-id=@prodPhotos.ProPhotoId 
        data-title=@prodPhotos.Title data-toggle="modal" href="#add-prodphoto-photo">Add Photo</a>
        
        <a class="badge badge-pill sponsor-modal-button" data-id=@sponsors.SponsorId 
        data-title=@sponsors.Name data-toggle="modal" href="#add-sponsor-photo">Add Photo</a>
        
        <a class="badge badge-pill cast-modal-button" data-id=@castMembers.CastMemberID 
        data-title=@castMembers.Name data-toggle="modal" href="#add-cast-photo">Add Photo</a>
        
The code snippet below contains the code for displaying the modal when the pill button is clicked and the forms responsible for uploading the photo data.
        
        @using (Html.BeginForm("UpdateProductionsWithoutPhotos", "Admin", FormMethod.Post,
        new { enctype = "multipart/form-data" }))
        {
            @Html.AntiForgeryToken()
            <div id="add-production-photo" class="modal fade">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h4 class="modal-title">Add photo for <span id="production-title"></span></h4>
                            <button type="button" class="close" data-dismiss="modal">&times;</button>
                        </div>

                        <div class="modal-body">
                            <form>
                                <div class="custom-file">
                                    <input type="file" id="production-image" class="custom-file-input"
                                    name="file" accept="image/*">
                                    <label class="custom-file-label" for="model-image">Choose file</label>
                                </div>
                                <input type="hidden" id="production-id" name="id" />
                            </form>
                        </div>

                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Cancel</button>
                            <button type="submit" class="btn btn-primary">Create</button>
                        </div>
                    </div>
                </div>
            </div>
        }

        @using (Html.BeginForm("UpdateProductionPhotosWithoutPhotos", "Admin", FormMethod.Post,
        new { enctype = "multipart/form-data" }))
        {
            @Html.AntiForgeryToken()
            <div id="add-prodphoto-photo" class="modal fade">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h4 class="modal-title">Add photo for <span id="prod-photo-title"></span></h4>
                            <button type="button" class="close" data-dismiss="modal">&times;</button>
                        </div>

                        <div class="modal-body">
                            <form>
                                <div class="custom-file">
                                    <input type="file" id="prod-photo-image" class="custom-file-input"
                                    name="file" accept="image/*">
                                    <label class="custom-file-label" for="model-image">Choose file</label>
                                </div>
                                <input type="hidden" id="prod-photo-id" name="id" />
                            </form>
                        </div>

                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Cancel</button>
                            <button type="submit" class="btn btn-primary">Create</button>
                        </div>
                    </div>
                </div>
            </div>
        }

        @using (Html.BeginForm("UpdateSponsorsWithoutPhotos", "Admin", FormMethod.Post,
        new { enctype = "multipart/form-data" }))
        {
            @Html.AntiForgeryToken()
            <div id="add-sponsor-photo" class="modal fade">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h4 class="modal-title">Add photo for <span id="sponsor-title"></span></h4>
                            <button type="button" class="close" data-dismiss="modal">&times;</button>
                        </div>

                        <div class="modal-body">
                            <form>
                                <div class="custom-file">
                                    <input type="file" id="sponsor-image" class="custom-file-input"
                                    name="file" accept="image/*">
                                    <label class="custom-file-label" for="model-image">Choose file</label>
                                </div>
                                <input type="hidden" id="sponsor-id" name="id" />
                            </form>
                        </div>

                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Cancel</button>
                            <button type="submit" class="btn btn-primary">Create</button>
                        </div>
                    </div>
                </div>
            </div>
        }

        @using (Html.BeginForm("UpdateCastMembersWithoutPhotos", "Admin", FormMethod.Post,
        new { enctype = "multipart/form-data" }))
        {
            @Html.AntiForgeryToken()
            <div id="add-cast-photo" class="modal fade">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h4 class="modal-title">Add photo for <span id="cast-title"></span></h4>
                            <button type="button" class="close" data-dismiss="modal">&times;</button>
                        </div>

                        <div class="modal-body">
                            <form>
                                <div class="custom-file">
                                    <input type="file" id="cast-image" class="custom-file-input"
                                    name="file" accept="image/*">
                                    <label class="custom-file-label" for="model-image">Choose file</label>
                                </div>
                                <input type="hidden" id="cast-id" name="id" />
                            </form>
                        </div>

                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Cancel</button>
                            <button type="submit" class="btn btn-primary">Create</button>
                        </div>
                    </div>
                </div>
            </div>
        }
        
The code snippet below contains the POST methods in the controller used to upload the photos.

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult UpdateCastMembersWithoutPhotos(int id, HttpPostedFileBase file)
        {
            CastMember castMember = db.CastMembers.Find(id);

            if (ModelState.IsValid)
            {
                if (castMember.PhotoId == null)
                {
                    castMember.PhotoId = PhotoController.CreatePhoto(file, castMember.Name);
                }

                db.Entry(castMember).State = EntityState.Modified;
                db.SaveChanges();

                return RedirectToAction("Dashboard");
            }
            return View(castMember);
        }
        
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult UpdateProductionsWithoutPhotos(int id, HttpPostedFileBase file)
        {
            Production production = db.Productions.Find(id);
            ProductionPhotos productionPhoto = new ProductionPhotos()
            {
                Production = production,
                Title = production.Title,
                Description = production.Title
            };
            productionPhoto.PhotoId = PhotoController.CreatePhoto(file, productionPhoto.Title);
            var admin = new AdminController();
            int photoUnavailable = admin.db.Photo.Where(p => p.Title == "Photo Unavailable").
            Select(p => p.PhotoId).FirstOrDefault();

            if (ModelState.IsValid)
            {
                if (production.DefaultPhoto == null || production.DefaultPhoto.PhotoId == photoUnavailable)
                {
                    production.DefaultPhoto = productionPhoto;
                }

                db.ProductionPhotos.Add(productionPhoto);
                db.SaveChanges();

                db.Entry(production).State = EntityState.Modified;
                db.SaveChanges();

                return RedirectToAction("Dashboard");
            }
            return View(productionPhoto);
        }
        
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult UpdateProductionPhotosWithoutPhotos(int id, HttpPostedFileBase file)
        {
            ProductionPhotos productionPhoto = db.ProductionPhotos.Find(id);
            var admin = new AdminController();
            int photoUnavailable = admin.db.Photo.Where(p => p.Title == "Photo Unavailable").
            Select(p => p.PhotoId).FirstOrDefault();

            if (ModelState.IsValid)
            {
                if (productionPhoto.PhotoId == null || productionPhoto.
                PhotoId == photoUnavailable && productionPhoto.Production != null)
                {
                    productionPhoto.PhotoId = PhotoController.CreatePhoto(file, productionPhoto.Title);
                }

                db.Entry(productionPhoto).State = EntityState.Modified;
                db.SaveChanges();

                return RedirectToAction("Dashboard");
            }
            return View(productionPhoto);
        }
        
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult UpdateSponsorsWithoutPhotos(int id, HttpPostedFileBase file)
        {
            Sponsor sponsor = db.Sponsors.Find(id);

            if (ModelState.IsValid)
            {
                if (sponsor.LogoId == null)
                {
                    sponsor.LogoId = PhotoController.CreatePhoto(file, sponsor.Name);
                }

                db.Entry(sponsor).State = EntityState.Modified;
                db.SaveChanges();

                return RedirectToAction("Dashboard");
            }
            return View(sponsor);
        }

Finally, the code snippet below contains the JQuery that passes data from the specified model without a photo in the database to the respective modal.
      
      @Scripts.Render("~/bundles/jquery")
      <script>
          $(".dashboard-subsection").hover(
              function () {
                  $(this).find(".dashboard-badges").show();
              },
              function () {
                  $(this).find(".dashboard-badges").hide();
              }
          );

          $(document).on("click", ".production-modal-button", function () {
              var productionId = $(this).data('id');
              var productionTitle = $(this).data('title');
              $("#production-id").val(productionId);
              $("#production-title").html(productionTitle);
          });

          $(document).on("click", ".prod-photo-modal-button", function () {
              var prodPhotoId = $(this).data('id');
              var prodPhotoTitle = $(this).data('title');
              $("#prod-photo-id").val(prodPhotoId);
              $("#prod-photo-title").html(prodPhotoTitle);
          });

          $(document).on("click", ".sponsor-modal-button", function () {
              var sponsorId = $(this).data('id');
              var sponsorTitle = $(this).data('title');
              $("#sponsor-id").val(sponsorId);
              $("#sponsor-title").html(sponsorTitle);
          });

          $(document).on("click", ".cast-modal-button", function () {
              var castId = $(this).data('id');
              var castTitle = $(this).data('title');
              $("#cast-id").val(castId);
              $("#cast-title").html(castTitle);
          });
          
          $(".custom-file-input").on("change", function () {
              var fileName = $(this).val().split("\\").pop();
              $(this).siblings(".custom-file-label").addClass("selected").html(fileName);
          });
      </script>

This concludes my work with Prosper IT Consulting. All in all, it was a very fruitful endeavor and I look forward to future work in this field.
