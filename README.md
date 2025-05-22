<?php

namespace App\Controller;

use App\Entity\Appointment;
use App\Form\AppointmentType;
use App\Repository\AppointmentRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

final class SpecialistController extends AbstractController
{
    #[Route('/specialist', name: 'app_specialist_home')]
    public function index(AppointmentRepository $appointmentRepository): Response
    {
        $user = $this->getUser();
        $appointments = $appointmentRepository->findBy(['specialist' => $user]);
        return $this->render('specialist/index.html.twig', [
            'appointments' => $appointments,
        ]);
        
    }

    #[Route('/specialist/appointment', name: 'app_appointment_new')]
    public function new(Request $request, EntityManagerInterface $entityManager): Response
    {
        $appointment = new Appointment();
        $appointment->setSpecialist($this->getUser());
        
        $form = $this->createForm(AppointmentType::class, $appointment);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->persist($appointment);
            $entityManager->flush();
            $this->addFlash("success", "Afspraak succesvol toegevoegd");
            return $this->redirectToRoute('app_specialist_home');
        }

        return $this->render('specialist/new.html.twig', [
            'form' => $form,
        ]);
    }
    #[Route('/specialist/appointment/edit/{id}', name: 'app_appointment_update')]
    public function edit(EntityManagerInterface $entityManager,Request $request, Appointment $appointment): Response
    {
        $form = $this->createForm(AppointmentType::class, $appointment);
        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->persist($appointment);
            $entityManager->flush();
            $this->addFlash("success", "Afspraak succesvol aangepast");
            return $this->redirectToRoute('app_specialist_home', ['appointment' => $appointment->getId()]);
        }
        return $this->render('specialist/new.html.twig', [
            'form' => $form,
        ]);
    }

    #[Route('/specialist/delete/{id}', name: 'app_appointment_delete')]
    public function delete(EntityManagerInterface $entityManager, Appointment $appointment): Response {
        $entityManager->remove($appointment);
        $entityManager->flush();
        $this->addFlash('success', 'Afspraak is verwijderd');
        return $this->redirectToRoute('app_specialist_home');
    }


//    #[Route('/smartphone/show/{id}', name: 'app_show_smartphone')]
//    public function show(Smartphone $smartphone, EntityManagerInterface $entityManager, int $id): Response
//    {
//
//        return $this->render('smartphone/show.html.twig', [
//            'smartphone' => $entityManager->getRepository(Smartphone::class)->find($id),
//        ]);
//    }

}

<!-- Patient -->

<?php

namespace App\Controller;

use App\Entity\Appointment;
use App\Form\AppointmentType;
use App\Repository\AppointmentRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

final class PatientController extends AbstractController
{
    #[Route('/patient', name: 'app_patient_home')]
    public function index(AppointmentRepository $appointmentRepository): Response
    {
        $user = $this->getUser();
        $appointments = $appointmentRepository->findBy(['patient' => $user]);

        return $this->render('patient/index.html.twig', [
            'appointments' => $appointments,
        ]);
    }

    #[Route('/patient/appointment', name: 'app_patient_new')]
    public function new(Request $request, EntityManagerInterface $entityManager): Response
    {
        $appointment = new Appointment();
        $appointment->setPatient($this->getUser());
        $form = $this->createForm(AppointmentType::class, $appointment);
        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->persist($appointment);
            $entityManager->flush();
            $this->addFlash("success", "Afspraak succesvol toegevoegd");
            return $this->redirectToRoute('app_patient_home');
        }

        return $this->render('patient/new.html.twig', [
            'form' => $form,
        ]);

    }

}


<!-- HomeController -->

<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

final class HomeController extends AbstractController
{
    #[Route('/', name: 'app_home')]
    public function index(): Response
    {
        if($this->isGranted('ROLE_Admin')) {
//           dd("test 1");
            return $this->redirectToRoute('app_admin');
        }
//        dd("test ali");
//        dd($this->getUser()->getRoles());
        if($this->isGranted('ROLE_SPECIALIST')) {
//           dd("test 1");
            return $this->redirectToRoute('app_specialist_home');

        }

        if($this->isGranted('ROLE_PATIENT')) {
//            dd("test 2");
            return $this->redirectToRoute('app_patient_home');
        }
//        dd("test 3");
        return $this->render('home/index.html.twig', [
            'controller_name' => 'HomeController',
        ]);
    }
}

<!-- Patient Role Twig template index -->
{% extends 'base.html.twig' %}

{% block title %}Mijn Afspraken{% endblock %}

{% block body %}
    {% include 'nav.html.twig' %}
<style>
    .example-wrapper { margin: 1em auto; max-width: 800px; width: 95%; font: 18px/1.5 sans-serif; }
    .example-wrapper code { background: #F5F5F5; padding: 2px 6px; }
</style>

    <div class="container-fluid">
        <div class="row">
            <div class="col-md-10">
                <h1>Welkom {{ app.user.firstName }} {{ app.user.lastName }}</h1>
                <h2>Mijn Afspraken</h2>

                <table class="table">
                    <thead>
                    <tr>
                        <th scope="col">Datum</th>
                        <th scope="col">Tijd</th>
                        <th scope="col">Specialist</th>
                        <th scope="col">Specialisme</th>
                        <th scope="col">Onderwerp</th>
                        <th scope="col">Problemen</th>
                        <th scope="col">Opmerkingen</th>
                    </tr>
                    </thead>
                    <tbody>
                    {% for appointment in appointments %}
                        <tr>
                            <td>{{ appointment.date|date('d-m-Y') }}</td>
                            <td>{{ appointment.time|date('H:i') }}</td>
                            <td>{{ appointment.specialist.firstName }} {{ appointment.specialist.lastName }}</td>
                            <td>{{ appointment.specialist.specialization }}</td>
                            <td>{{ appointment.subject }}</td>
                            <td>{{ appointment.problems }}</td>
                            <td>{{ appointment.discussed }}</td>
                        </tr>
                    {% else %}
                        <tr>
                            <td colspan="7">Geen afspraken gevonden</td>
                        </tr>
                    {% endfor %}
                    </tbody>
                </table>
                <a href="{{ path('app_patient_new') }}" class="btn btn-primary">Nieuwe Afspraak</a>
            </div>
        </div>
    </div>
{% endblock %}


<!-- Specialist Role index -->

{% extends 'base.html.twig' %}

{% block title %}Mijn Afspraken{% endblock %}

{% block body %}
    {% include 'nav.html.twig' %}
<style>
    .example-wrapper { margin: 1em auto; max-width: 800px; width: 95%; font: 18px/1.5 sans-serif; }
    .example-wrapper code { background: #F5F5F5; padding: 2px 6px; }
</style>

    <div class="container-fluid">
        <div class="row">
            <div class="col-md-10">
                <h1>Welkom dr. {{ app.user.firstName }} {{ app.user.lastName }}</h1>
                <h2>Mijn Afspraken</h2>

                <table class="table">
                    <thead>
                    <tr>
                        <th scope="col">Datum</th>
                        <th scope="col">Tijd</th>
                        <th scope="col">Patiënt</th>
                        <th scope="col">Onderwerp</th>
                        <th scope="col">Problemen</th>
                        <th scope="col">Opmerkingen</th>
                        <th scope="col">Update</th>
                        <th scope="col">Delete</th>
                    </tr>
                    </thead>
                    <tbody>
                    {% for appointment in appointments %}
                        <tr>
                            <td>{{ appointment.date|date('d-m-Y') }}</td>
                            <td>{{ appointment.time|date('H:i') }}</td>
                            <td>{{ appointment.patient.firstName }} {{ appointment.patient.lastName }}</td>
                            <td>{{ appointment.subject }}</td>
                            <td>{{ appointment.problems }}</td>
                            <td>{{ appointment.discussed }}</td>
                            <td><a href="{{ path('app_appointment_update', {'id':appointment.id}) }}" class="btn btn-warning">update</a></td>
                            <td><a href="{{ path('app_appointment_delete', {'id':appointment.id}) }}" class="btn btn-danger">delete</a></td>
                        </tr>
                    {% else %}
                        <tr>
                            <td colspan="6">Geen afspraken gevonden</td>
                        </tr>
                    {% endfor %}
                    </tbody>
                </table>
                <a href="{{ path('app_appointment_new') }}" class="btn btn-primary">Nieuwe Afspraak</a>
            </div>
        </div>
    </div>
{% endblock %}


<!-- Relaties ManyToOne -->

specialist: ManyToOne → User
php bin/console make:entity Appointment

New property name: specialist
Field type: relation
Related class: User
Type of relation: ManyToOne
Nullable: yes
Inverse relation: no

إضافة العلاقات إلى Appointment
patient: ManyToOne → User

php bin/console make:entity Appointment
New property name: patient
Field type: relation
Related class: User
Type of relation: ManyToOne
Nullable: yes
Inverse relation: no

3. إنشاء العلاقات مع User
للعلاقة مع المريض (patient_id):
php bin/console make:entity Appointment
New property name (e.g. isPublished): patient
Field type (enter ? to see all types) [string]: relation
What class should this entity be related to?: User
What type of relationship is this?:
  [0] ManyToOne
  [1] OneToMany
  [2] ManyToMany
  [3] OneToOne
 > 0
Is the Appointment.patient property allowed to be null (nullable)? (yes/no) [yes]:
 > yes
Do you want to add a new property to User so that you can access/update Appointment objects from it - e.g. $user->getAppointments()? (yes/no) [no]:
 > no
